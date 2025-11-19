# Hand-Safe
Projeto Iot- Universidade Presbiteriana Mackenzie
#include <ESP8266WiFi.h>
#include <PubSubClient.h>
#include <Servo.h>

Servo servo;
const int pirPin = D5;

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "test.mosquitto.org";

WiFiClient espClient;
PubSubClient client(espClient);

void reconnect() {
  while (!client.connected()) {
    if (client.connect("Biazinha-Handsafe")) {
      client.subscribe("biazinha/teste");
    } else {
      delay(2000);
    }
  }
}

void callback(char* topic, byte* payload, unsigned int length) {
  if (payload[0] == '1') {
    servo.write(90);
  } else {
    servo.write(0);
  }
}

void setup() {
  servo.attach(D6);
  pinMode(pirPin, INPUT);

  WiFi.begin(ssid, password);

  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  
  client.loop();

  int motion = digitalRead(pirPin);
  
  if (motion == HIGH) {
    client.publish("biazinha/teste", "1");
    servo.write(90);
    delay(1500);
    servo.write(0);
  } else {
    client.publish("biazinha/teste", "0");
  }

  delay(200);
}
