# Lecture 03 Task 1 - Furkan Atalay 36447

## 1. Wokwi Project Link
- [Wokwi Simulation Project](https://wokwi.com/projects/30556959398609473)

## 2. Screenshots
- Circuit: `circuit.png`
- Alerts Dashboard: `edp-alerts.png`

## 3. Explanation
In this task, the events originate from physical user interactions with a push button connected to the ESP32 development board. When the button is pressed, the circuit pulls the GPIO pin from HIGH to LOW, creating an input trigger. To ensure each alert is sent only once per press, software debouncing logic utilizing `millis()` is implemented, which filters out electrical noise and transient states. The digital signal travels from the push button through the jumper wire directly into GPIO 4 of the ESP32. Even though HTTP requests over Wi-Fi can be blocking operations, the asynchronous nature of the main event loop and proper state management ensures that inputs are tracked accurately.

## 4. Serial Monitor Output
```text
load:0x3fff0030,len:1156
load:0x40078000,len:11456
ho 0 tail 12 room 4
load:0x40080400,len:2972
entry 0x400805dc
....
WiFi connected.
[Event] Button pressed!
[HTTP] Response Code: 200
#include <WiFi.h>
#include <WiFiClientSecure.h>
#include <HTTPClient.h>

const char* ssid = "Wokwi-GUEST";
const char* password = "";

const int buttonPin = 4;

unsigned long lastDebounceTime = 0;
unsigned long debounceDelay = 50;
int lastButtonState = HIGH;

void setup() {
  Serial.begin(115200);
  pinMode(buttonPin, INPUT_PULLUP);

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi connected.");
}

void loop() {
  int reading = digitalRead(buttonPin);

  if (reading != lastButtonState) {
    lastDebounceTime = millis();
  }

  if ((millis() - lastDebounceTime) > debounceDelay) {
    if (reading == LOW && lastButtonState == HIGH) {
      Serial.println("[Event] Button pressed!");
      sendAlert();
    }
  }
  lastButtonState = reading;
}

void sendAlert() {
  if (WiFi.status() == WL_CONNECTED) {
    WiFiClientSecure client;
    client.setInsecure();

    HTTPClient http;
    http.begin(client, "[https://edp-alerts.codewithpeter.com/alerts](https://edp-alerts.codewithpeter.com/alerts)");
    http.addHeader("Content-Type", "application/json");

    String httpRequestData = "{\"event\": \"fire\", \"student\": \"Furkan Atalay 36447\"}";
    int httpResponseCode = http.POST(httpRequestData);

    Serial.print("[HTTP] Response Code: ");
    Serial.println(httpResponseCode);
    http.end();
  }
}
