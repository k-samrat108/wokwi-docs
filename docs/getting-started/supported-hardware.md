#include <ESP8266WiFi.h>
#include <espnow.h>

// HC-SR04 pins
#define TRIG_PIN D5
#define ECHO_PIN D6

// Receiver MAC Address (dummy for now — replace with your actual receiver MAC)
uint8_t receiverMac[] = {0x24, 0x6F, 0x28, 0xAB, 0xCD, 0xEF};

void setup() {
  Serial.begin(115200);

  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);

  WiFi.mode(WIFI_STA);
  WiFi.disconnect();  // No internet needed

  if (esp_now_init() != 0) {
    Serial.println("ESP-NOW init failed");
    return;
  }

  esp_now_set_self_role(ESP_NOW_ROLE_CONTROLLER);
  esp_now_add_peer(receiverMac, ESP_NOW_ROLE_SLAVE, 1, NULL, 0);
}

void loop() {
  long duration;
  float distance_cm;

  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);

  duration = pulseIn(ECHO_PIN, HIGH);
  distance_cm = duration * 0.034 / 2;

  // Convert to integer before sending
  int level = (int)distance_cm;
  esp_now_send(receiverMac, (uint8_t *)&level, sizeof(level));

  Serial.print("Distance: ");
  Serial.print(level);
  Serial.println(" cm");

  delay(2000);  // Send every 2 seconds
}
