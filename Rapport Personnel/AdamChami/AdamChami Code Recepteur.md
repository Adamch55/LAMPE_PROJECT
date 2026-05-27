code receveur:
 "#include <SPI.h>
#include <LoRa.h>

// Broches matérielles pour la carte UCA
const int csPin = 10;
const int resetPin = 8;
const int irqPin = 6;

unsigned long dernierScan = 0;

void setup() {
  Serial.begin(115200);
  while (!Serial);

  Serial.println("Tentative d'initialisation du LoRa (Recepteur UCA)...");
 
  // Application des broches matérielles
  LoRa.setPins(csPin, resetPin, irqPin);

  if (!LoRa.begin(868E6)) {
    Serial.println("Echec LoRa ! Verifiez les constantes csPin/resetPin/irqPin.");
    while (1);
  }
 
  LoRa.setGain(6); // Correction ici : Gain de réception max configuré correctement
  Serial.println("Recepteur UCA pret. En attente de paquets...");
}

void loop() {
  int packetSize = LoRa.parsePacket();
 
  if (packetSize) {
    String messageRecu = "";
    while (LoRa.available()) {
      messageRecu += (char)LoRa.read();
    }

    int valeurLuminosite = messageRecu.toInt();
    Serial.print("Paquet recu ! Luminosite : ");
    Serial.println(valeurLuminosite);
  }

  // Petit message de controle toutes les 5 secondes
  if (millis() - dernierScan > 5000) {
    Serial.println("... En attente de signal LoRa (868MHz) ...");
    dernierScan = millis();
  }
}
