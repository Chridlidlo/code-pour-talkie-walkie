//# code-pour-talkie-walkie
//voici le code
#include <SPI.h>
#include <nRF24L01.h>
#include <RF24.h>

RF24 radio(9, 10); // Configuration des broches CE et CSN
const byte address[6] = "00001"; // Adresse de communication

const int speakerPin = 6; // Broche du haut-parleur
const int microphonePin = A0; // Broche du microphone
const int redPin = 3; // Broche pour la LED rouge
const int greenPin = 5; // Broche pour la LED verte
const int bluePin = 6; // Broche pour la LED bleue

void setup() {
  Serial.begin(9600);
  radio.begin();
  radio.openWritingPipe(address);
  radio.openReadingPipe(1, address);
  radio.setPALevel(RF24_PA_MIN); // Puissance de transmission minimale
  radio.startListening();

  pinMode(speakerPin, OUTPUT);
  pinMode(redPin, OUTPUT);
  pinMode(greenPin, OUTPUT);
  pinMode(bluePin, OUTPUT);

  digitalWrite(redPin, HIGH); // Activer la LED rouge
}

void loop() {
  if (radio.available()) {
    char text[32] = ""; // Créez un tableau de caractères pour stocker le message
    radio.read(&text, sizeof(text)); // Lecture du message reçu depuis l'autre Arduino
    Serial.println("Message reçu: " + String(text)); // Afficher le message sur le moniteur série
    // Jouer le son reçu à travers le haut-parleur
    int soundFreq = atoi(text); // Convertir le texte en fréquence sonore
    tone(speakerPin, soundFreq); // Jouer la fréquence sonore
    delay(500);
    noTone(speakerPin); // Arrêter le son
    // Activer la LED verte pendant la réception d'un message
    digitalWrite(greenPin, HIGH);
    delay(1000);
    digitalWrite(greenPin, LOW);
  }
  
  if (Serial.available()) {
    char text[32] = ""; // Créez un tableau de caractères pour stocker le message
    Serial.readBytesUntil('\n', text, sizeof(text)); // Lire le message entrant depuis le moniteur série
    radio.stopListening(); // Arrête d'écouter pour pouvoir transmettre
    radio.write(&text, sizeof(text)); // Envoyer le message à l'autre Arduino
    radio.startListening(); // Revenir à l'écoute
    Serial.println("Message envoyé: " + String(text)); // Afficher le message envoyé sur le moniteur série
    // Capturer le son du microphone et transmettre la fréquence sonore
    int soundLevel = analogRead(microphonePin);
    if (soundLevel > 100) { // Condition pour éviter d'envoyer un message vide
      radio.stopListening(); // Arrête d'écouter pour pouvoir transmettre
      radio.write(String(soundLevel).c_str(), sizeof(soundLevel)); // Envoyer la fréquence sonore à l'autre Arduino
      radio.startListening(); // Revenir à l'écoute
      Serial.println("Sound Level: " + String(soundLevel)); // Afficher le niveau sonore sur le moniteur série
      delay(1000); // Attendez un court instant pour éviter la répétition des messages
      // Activer la LED bleue pendant l'envoi d'un message
      digitalWrite(bluePin, HIGH);
      delay(1000);
      digitalWrite(bluePin, LOW);
    }
  }
}
