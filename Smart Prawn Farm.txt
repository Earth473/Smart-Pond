#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET    -1  
#define OLED_ADDRESS   0x3C  

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);


#define ONE_WIRE_BUS 2
#define BTN_PIN 7
#define WATER_LV_SEN A0
#define RELAY_PIN 13
#define RELAY_PIN_2 12
#define RELAY_PIN_3 11
#define RELAY_PIN_4 10
#define PH_SENSOR_PIN A1

int btn_state;
int wrt_lv_val;
bool relay_state;
bool relay_state_2;
bool relay_state_3;
bool relay_state_4;
bool servoActivated = false;

#include <RTC.h>
#include "thingProperties.h"
#include <OneWire.h>
#include <DallasTemperature.h>
#include <Servo.h>

OneWire oneWire(ONE_WIRE_BUS);
DallasTemperature sensors(&oneWire);
Servo myservo;

void setup() {
  pinMode(RELAY_PIN, OUTPUT);
  pinMode(RELAY_PIN_2, OUTPUT);
  pinMode(RELAY_PIN_3, OUTPUT);
  pinMode(RELAY_PIN_4, OUTPUT);
  pinMode(BTN_PIN, INPUT);
  myservo.attach(9);

  Serial.begin(9600);
  delay(1000);
  Serial.println("Dallas Temperature IC Control Library");
  sensors.begin();
 
  initProperties();
  ArduinoCloud.begin(ArduinoIoTPreferredConnection);
  setDebugMessageLevel(2);
  ArduinoCloud.printDebugInfo();

 
  display.begin(SSD1306_SWITCHCAPVCC, OLED_ADDRESS); 
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
}

void loop() {
  ArduinoCloud.update();

  wrt_lv_val = analogRead(WATER_LV_SEN);
  btn_state = digitalRead(BTN_PIN);
 
 
  float phValue = readPH();

  
  if (btn_state == 1) {
    controlRelaysWithButtonAndSensor();
  } else {
    autowaterpump();
    controlRelaysFromCloud();
  }

  
    if (relay_state == false) {
        onServo1Change(); 
    }
  
  if (!servoActivated) {
    onServo1Change();
    servoActivated = true; 
  } else if (servo_1 == false) {
    servoActivated = false; 
  }

  
  Serial.print("Water Level Value: ");
  Serial.println(wrt_lv_val);
  sensors.requestTemperatures(); 
  Serial.print("Temperature is: ");
  Serial.print(sensors.getTempCByIndex(0)); 
  Serial.println(" *C");
  temp_value = sensors.getTempCByIndex(0);
 
  display.clearDisplay(); 
  display.setCursor(0, 0); 
  display.print("Water Level: ");
  display.print(wrt_lv_val);
  display.println(" units");

  display.print("Temperature: ");
  display.print(sensors.getTempCByIndex(0));
  display.println(" *C");

  display.print("pH Value: ");
  display.print(phValue);

  display.display(); 
  delay(500);
}

float readPH() {
  int sensorValue = analogRead(PH_SENSOR_PIN);
  float voltage = sensorValue * (5.0 / 1023.0); 
  float phValue = 7 + ((3.9 - voltage) / 0.18); 
 return phValue;
}


void controlRelaysWithButtonAndSensor() {
  if (wrt_lv_val < 100) {
    digitalWrite(RELAY_PIN_2, LOW);
    relay_state_2 = false;
  } else {
    digitalWrite(RELAY_PIN_2, HIGH);
    relay_state_2 = true;
  }
}

void autowaterpump() {
  if (wrt_lv_val > 100) {
    digitalWrite(RELAY_PIN, LOW);
    relay_state = false;
  } else {
    digitalWrite(RELAY_PIN, HIGH);
    relay_state = true;
  }
}

void controlRelaysFromCloud() {
  digitalWrite(RELAY_PIN_2, relay_2 ? LOW : HIGH);
  digitalWrite(RELAY_PIN_3, relay_3 ? LOW : HIGH);
  digitalWrite(RELAY_PIN_4, relay_4 ? LOW : HIGH);
}

void onRelay2Change() {
  digitalWrite(RELAY_PIN_2, relay_2 ? LOW : HIGH);
}

void onRelay3Change() {
  digitalWrite(RELAY_PIN_3, relay_3 ? LOW : HIGH);
}

void onRelay4Change() {
  digitalWrite(RELAY_PIN_4, relay_4 ? LOW : HIGH);
}

void onTempValueChange()  {
 
}

void onServo1Change()  {
  if (servo_1 == true) {
    myservo.write(0); 
    delay(500);
    myservo.write(180); 
    delay(500);
    myservo.write(0); 
  } else {
    myservo.write(0); 
  }
}