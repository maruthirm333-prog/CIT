2 options whether build the hardware or software so we decided to build the soft so : 


# CIT
DATA
PHASE 1 — Yard node (ESP32 #1)
Load cell → HX711
Wire	HX711
Red	E+
Black	E−
White	A−
Yellow	A+

B+ and B− empty.

HX711 → ESP32
HX711	ESP32
VCC	5V (VIN)
GND	GND
DT	GPIO 25
SCK	GPIO 26
SD card → ESP32
SD	ESP32
VCC	5V
GND	GND
MISO	GPIO 19
MOSI	GPIO 23
SCK	GPIO 18
CS	GPIO 5

Card must be FAT32.

Power

Power bank via USB.

Your numbers — keep these on paper
CAL FACTOR    = -218.5005
START WEIGHT  = 242.7 g
START TIME    = 16:26, 29 Oct
TARGET        = 6% moisture
Code — currently running
cpp
#include "HX711.h"
#include <SPI.h>
#include <SD.h>

HX711 scale;
const int CS_PIN = 5;
unsigned long count = 0;

void setup() {
  Serial.begin(115200);
  scale.begin(25, 26);
  delay(2000);
  scale.set_scale(-218.5005);
  scale.tare(20);

  if (!SD.begin(CS_PIN)) {
    Serial.println("SD FAIL");
    while (1) delay(1000);
  }
  Serial.println("SD OK");

  File f = SD.open("/dry.csv", FILE_APPEND);
  if (f) { f.println("minute,grams"); f.close(); }
  Serial.println("Logging started");
}

void loop() {
  float w = scale.get_units(20);

  File f = SD.open("/dry.csv", FILE_APPEND);
  if (f) {
    f.print(count); f.print(",");
    f.println(w, 1);
    f.close();
  }

  Serial.print(count); Serial.print(" min  ");
  Serial.print(w, 1);  Serial.println(" g");

  count++;
  delay(60000);
}
Code — with moisture % (for the demo)
cpp
#include "HX711.h"
#include <SPI.h>
#include <SD.h>

HX711 scale;
const int CS_PIN = 5;

const float START_WEIGHT = 242.7;
const float START_MC     = 0.50;
const float TARGET_MC    = 6.0;

float dryMatter;
unsigned long count = 0;

void setup() {
  Serial.begin(115200);
  scale.begin(25, 26);
  delay(2000);
  scale.set_scale(-218.5005);
  scale.tare(20);

  dryMatter = START_WEIGHT * (1.0 - START_MC);

  if (!SD.begin(CS_PIN)) {
    Serial.println("SD FAIL");
    while (1) delay(1000);
  }

  File f = SD.open("/dry.csv", FILE_APPEND);
  if (f) { f.println("minute,grams,moisture"); f.close(); }
  Serial.println("Logging started");
}

void loop() {
  float w  = scale.get_units(20);
  float mc = (w - dryMatter) / w * 100.0;
  if (w <= 0) mc = 0;

  File f = SD.open("/dry.csv", FILE_APPEND);
  if (f) {
    f.print(count); f.print(",");
    f.print(w, 1);  f.print(",");
    f.println(mc, 1);
    f.close();
  }

  Serial.print(count); Serial.print(" min  ");
  Serial.print(w, 1);  Serial.print(" g  ");
  Serial.print(mc, 1); Serial.print(" %  ");
  Serial.println(mc <= TARGET_MC ? "STOP DRYING" : "CONTINUE");

  count++;
  delay(60000);
}
Rules for the running node
Tare with the empty disc, then add copra
Shade the scale body, copra in sun
Tape it down
Don’t touch it once running
