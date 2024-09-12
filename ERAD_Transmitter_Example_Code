#include <esp_now.h>
#include <WiFi.h>

// Define the potentiometer pin
#define POTENTIOMETER_PIN 4 // Feel free to change this to whichever pin you'd like

// Structure to send data
typedef struct struct_message {
  int potValue;
} struct_message;

// Create a struct_message to hold the potentiometer value
struct_message potData;

// MAC address of the receiver (replace with the actual MAC address of the receiver)
uint8_t receiverAddress[] = {0x34, 0x85, 0x18, 0xCD, 0x5B, 0xA6}; // CHANGE THIS TO YOUR RECIEVER'S MAC!

// Callback when data is sent
void onDataSent(const uint8_t *mac_addr, esp_now_send_status_t status) {
  //Serial.print("Last Packet Send Status: ");
  //Serial.println(status == ESP_NOW_SEND_SUCCESS ? "Delivery Success" : "Delivery Fail");
}

void setup() {
  // Initialize Serial Monitor
  Serial.begin(115200);

  // Set device as a Wi-Fi station
  WiFi.mode(WIFI_STA);
  Serial.println("ESP-NOW Sender");

  // Initialize ESP-NOW
  if (esp_now_init() != ESP_OK) {
    Serial.println("Error initializing ESP-NOW");
    return;
  }

  // Register the send callback
  esp_now_register_send_cb(onDataSent);

  // Add the peer (receiver)
  esp_now_peer_info_t peerInfo;
  memcpy(peerInfo.peer_addr, receiverAddress, 6);
  peerInfo.channel = 0;  
  peerInfo.encrypt = false;

  if (esp_now_add_peer(&peerInfo) != ESP_OK) {
    Serial.println("Failed to add peer");
    return;
  }
}

void loop() {
  // Read potentiometer value
  potData.potValue = analogRead(POTENTIOMETER_PIN);

  // Send potentiometer value to the receiver
  esp_err_t result = esp_now_send(receiverAddress, (uint8_t *)&potData, sizeof(potData));

  // Print status of send operation
  if (result == ESP_OK) {
    //Serial.print("Sent: ");
    //Serial.println(potData.potValue);
  } else {
    //Serial.println("Error sending the data");
  }

  // Wait for 10ms before sending the next value
  delay(10);
}
