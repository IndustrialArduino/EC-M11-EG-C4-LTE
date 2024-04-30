#include <SPI.h>
#include "Adafruit_MAX31855.h"

#define MAXDO  19
#define MAXCS   5
#define MAXCLK 18
#define OUTPUT1 21

Adafruit_MAX31855 thermocouple(MAXCLK, MAXCS, MAXDO);

#define MODEM_RESET 32
#define MODEM_FLIGHT 22
#define MODEM_RX 26
#define MODEM_TX 25

long timer1;

void setup() {
  Serial.begin(115200);
  pinMode(MODEM_FLIGHT, OUTPUT);
  pinMode(MODEM_RESET, OUTPUT);
  digitalWrite(MODEM_FLIGHT, HIGH);

  pinMode(OUTPUT1, OUTPUT);
  digitalWrite(OUTPUT1, HIGH);
  delay(1000);
  digitalWrite(OUTPUT1, LOW);

  Serial.println("MAX31855 test");
  delay(500);
  Serial.print("Initializing sensor...");
  if (!thermocouple.begin()) {
    Serial.println("ERROR.");
    while (1) delay(10);
  }
  Serial.println("DONE.");
  
  MODEM_RESET_CYC();
  delay(2000);
  Serial2.begin(115200, SERIAL_8N1, MODEM_RX, MODEM_TX);
  Serial.println("SIM AT START >>>>>>>>>>>>>>");
  delay(2000);
  Serial2.println("AT");
  delay(2000);
  Serial2.println("AT+CPIN?");
  delay(2000);
  Serial2.println("AT+CNMP?");
}

void loop() {
  // Modem part
  delay(3000);
  timer1 = millis();
  Serial2.println("AT");
  while (millis() < timer1 + 10000) {
    while (Serial2.available()) {
      int inByte = Serial2.read();
      Serial.write(inByte);
    }
  }

  timer1 = millis();
  Serial2.println("AT+CPIN?");
  while (millis() < timer1 + 10000) {
    while (Serial2.available()) {
      int inByte = Serial2.read();
      Serial.write(inByte);
    }
  }

  // Thermocouple sensor part
  Serial.print("Internal Temp = ");
  Serial.println(thermocouple.readInternal());

  double c = thermocouple.readCelsius();
  if (isnan(c)) {
    Serial.println("Thermocouple fault(s) detected!");
    uint8_t e = thermocouple.readError();
    if (e & MAX31855_FAULT_OPEN) Serial.println("FAULT: Thermocouple is open - no connections.");
    if (e & MAX31855_FAULT_SHORT_GND) Serial.println("FAULT: Thermocouple is short-circuited to GND.");
    if (e & MAX31855_FAULT_SHORT_VCC) Serial.println("FAULT: Thermocouple is short-circuited to VCC.");
  } else {
    Serial.print("C = ");
    Serial.println(c);
  }

  Serial.println("");
  delay(1000);
}

void MODEM_RESET_CYC() {
  digitalWrite(MODEM_RESET, HIGH);
  delay(1000);
  digitalWrite(MODEM_RESET, LOW);
  delay(1000);
  digitalWrite(MODEM_RESET, HIGH);
}
