code emmeteur: 

"

#include <Wire.h>

#include <Adafruit_GFX.h>

#include <Adafruit_SSD1306.h>

#include "Adafruit_LTR329_LTR303.h"

#include <SPI.h>

#include <LoRa.h>


#define SCREEN_WIDTH 128

#define SCREEN_HEIGHT 64

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

Adafruit_LTR303 ltr = Adafruit_LTR303();


// Pins pour les LED

const int pinRed   = A1; 

const int pinBlue  = A2;

const int pinWhite = A3; 


// CONFIGURATION DES BROCHES LORA DE LA CARTE UCA

// Si l'initialisation échoue, on testera resetPin = 9 et irqPin = 2

const int csPin = 10;

const int resetPin = 8;

const int irqPin = 6;


void setup() {

  Serial.begin(115200);


  // 1. Initialisation des LED

  pinMode(pinRed,   OUTPUT);

  pinMode(pinBlue,  OUTPUT);

  pinMode(pinWhite, OUTPUT);


  // 2. Démarrage de l'I2C (A4/A5)

  Wire.begin();

  delay(500);


  // 3. Initialisation de l'OLED

  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) { 

    display.begin(SSD1306_SWITCHCAPVCC, 0x3D);

  }


  // 4. Force l'activation de l'écran

  display.ssd1306_command(SSD1306_DISPLAYON); 

  display.clearDisplay();

  display.display();


  // 5. Initialisation du capteur LTR303

  if (!ltr.begin()) {

    Serial.println(F("LTR303 non trouve !"));

  }


  ltr.setGain(LTR3XX_GAIN_1);

  ltr.setIntegrationTime(LTR3XX_INTEGTIME_100);

  ltr.setMeasurementRate(LTR3XX_MEASRATE_200);


  // 6. Initialisation Forcée du LoRa avec les pins définies

  LoRa.setPins(csPin, resetPin, irqPin);

  

  Serial.println("Tentative de demarrage du LoRa...");

  if (!LoRa.begin(868E6)) {

    Serial.println("Echec LoRa ! Verifiez les constantes csPin/resetPin/irqPin.");

    while (1);

  }

  

  LoRa.setTxPower(20); // Puissance max

  Serial.println(F("Systeme Emetteur UCA Pret"));

}


void loop() {

  uint16_t visible, ir;

  

  if (ltr.readBothChannels(visible, ir)) {

    // Logique des LED

    digitalWrite(pinWhite, (visible < 100));

    digitalWrite(pinBlue,  (visible >= 100 && visible < 150));

    digitalWrite(pinRed,   (visible >= 150 && visible <= 250));


    // Affichage OLED

    display.clearDisplay();

    display.setTextColor(SSD1306_WHITE);

    display.setTextSize(1);

    display.setCursor(0, 0);

    display.println(F("LIGHT SENSOR"));

    display.drawLine(0, 10, 128, 10, SSD1306_WHITE);


    display.setTextSize(1);

    display.setCursor(0, 18);

    display.print(F("Raw Value:"));


    display.setTextSize(2);

    display.setCursor(0, 32);

    display.print(visible);


    int barWidth = map(visible, 0, 2000, 0, 128);

    display.fillRect(0, 58, constrain(barWidth, 0, 128), 6, SSD1306_WHITE);

    display.display();


    // Envoi LoRa

    Serial.print("Envoi LoRa : ");

    Serial.println(visible);


    LoRa.beginPacket();

    LoRa.print(visible); 

    LoRa.endPacket();

  }

  

  delay(150);

}
