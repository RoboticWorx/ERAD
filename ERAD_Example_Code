#include <SimpleFOC.h>
#include <esp_now.h>
#include <WiFi.h>

// Define the BLDC motor pins
#define BLDC_PWM_UH_GPIO 12
#define BLDC_PWM_UL_GPIO 11
#define BLDC_PWM_VH_GPIO 10
#define BLDC_PWM_VL_GPIO 9
#define BLDC_PWM_WH_GPIO 8
#define BLDC_PWM_WL_GPIO 18

#define DRV0_CSN_PIN 48
#define DRV1_SCK_PIN 47

#define HALLU_PIN 6
#define HALLV_PIN 7
#define HALLW_PIN 15

#define ENABLE_PIN 38

#define POLE_PAIRS 7 // Number of permanent magnets in motor divided by 2

// Structure to receive data
typedef struct struct_message {
  int potValue;
} struct_message;

// Create a struct_message to hold the incoming data
struct_message incomingData;

// Callback function that runs when data is received
void onDataRecv(const esp_now_recv_info *info, const uint8_t *data, int len) {
  memcpy(&incomingData, data, sizeof(incomingData));  // Copy the received data into the struct
  //Serial.print("Received potentiometer value: ");
  //Serial.println(incomingData.potValue);  // Print the received potentiometer value
}

// Create BLDCMotor and BLDCDriver6PWM objects
BLDCMotor motor = BLDCMotor(POLE_PAIRS);
BLDCDriver6PWM driver = BLDCDriver6PWM(BLDC_PWM_UH_GPIO, BLDC_PWM_UL_GPIO, BLDC_PWM_VH_GPIO, BLDC_PWM_VL_GPIO, BLDC_PWM_WH_GPIO, BLDC_PWM_WL_GPIO);

// hall sensor instance
HallSensor sensor = HallSensor(HALLU_PIN, HALLV_PIN, HALLW_PIN, POLE_PAIRS);

// Interrupt routine intialisation
// channel A and B callbacks
void doA(){sensor.handleA();}
void doB(){sensor.handleB();}
void doC(){sensor.handleC();}

// velocity set point variable
float max_velocity = 100; // Feel free to adjust this

// instantiate the commander
Commander command = Commander(Serial);
void doTarget(char* cmd) { command.scalar(&max_velocity, cmd); }

void setup() {

  pinMode(ENABLE_PIN, OUTPUT);
  pinMode(DRV0_CSN_PIN, OUTPUT);
  pinMode(DRV1_SCK_PIN, OUTPUT);

  digitalWrite(DRV1_SCK_PIN, LOW);
  digitalWrite(DRV0_CSN_PIN, HIGH);
  
  digitalWrite(ENABLE_PIN, LOW);
  delay(250);
  digitalWrite(ENABLE_PIN, HIGH);

  // use monitoring with serial 
  Serial.begin(115200);
  // enable more verbose output for debugging
  // comment out if not needed
  //SimpleFOCDebug::enable(&Serial);

  // Set device as a Wi-Fi station
  WiFi.mode(WIFI_STA);
  Serial.println("ESP-NOW Receiver");

  // Initialize ESP-NOW
  if (esp_now_init() != ESP_OK) {
    Serial.println("Error initializing ESP-NOW");
    return;
  }

  // Register the receive callback
  esp_now_register_recv_cb(onDataRecv);

  // initialize sensor sensor hardware
  sensor.init();
  sensor.enableInterrupts(doA, doB, doC);

  // link the motor to the sensor
  motor.linkSensor(&sensor);

  // driver config
  // power supply voltage [V]
  driver.voltage_power_supply = 24;
  driver.init();
  // link the motor and the driver
  motor.linkDriver(&driver);

  // aligning voltage [V]
  motor.voltage_sensor_align = 2; // Change if needed

  // set motion control loop to be used
  motor.controller = MotionControlType::velocity;

  // contoller configuration
  // default parameters in defaults.h

  // velocity PI controller parameters
  motor.PID_velocity.P = 0.2f;
  motor.PID_velocity.I = 2;
  motor.PID_velocity.D = 0;
  // default voltage_power_supply
  motor.voltage_limit = 2; // Change if needed
  // jerk control using voltage voltage ramp
  // default value is 300 volts per sec  ~ 0.3V per millisecond
  motor.PID_velocity.output_ramp = 1000;

  // velocity low pass filtering time constant
  motor.LPF_velocity.Tf = 0.01f;

  // comment out if not needed
  //motor.useMonitoring(Serial);

  // initialize motor
  motor.init();
  // align sensor and start FOC
  motor.initFOC();

  // add target command T
  command.add('T', doTarget, "target voltage");

  Serial.println(F("Motor ready."));
  Serial.println(F("Set the target velocity using serial terminal:"));
  _delay(500);
}


void loop() {
  int velocity = map(incomingData.potValue, 0, 4096, -max_velocity, max_velocity);

  // main FOC algorithm function
  // the faster you run this function the better
  // Arduino UNO loop  ~1kHz
  // Bluepill loop ~10kHz
  motor.loopFOC();

  // Motion control function
  // velocity, position or voltage (defined in motor.controller)
  // this function can be run at much lower frequency than loopFOC() function
  // You can also use motor.move() and set the motor.target in the code
  motor.move(velocity);

  // function intended to be used with serial plotter to monitor motor variables
  // significantly slowing the execution down!!!!
  // motor.monitor();

  // user communication
  command.run();
}
