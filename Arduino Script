#include <DHT.h>

#define DHTPIN 2       // DHT Sensor an Pin 2
#define DHTTYPE DHT11  // DHT11 Sensor

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(9600);
  dht.begin();
}

void loop() {
  float h = dht.readHumidity();
  float t = dht.readTemperature();

  // Temperatur und Luftfeuchtigkeit über die serielle Schnittstelle an den Raspberry Pi senden
  if (isnan(h) || isnan(t)) {
    Serial.println("Fehler beim Lesen des DHT11 Sensors");
    return;
  }

  // Lese Lichtwert vom LDR (Beispielwert)
  int light = analogRead(A0);

  // Daten senden
  Serial.print("T:");
  Serial.print(t);  // Temperatur
  Serial.print(",H:");
  Serial.print(h);  // Luftfeuchtigkeit
  Serial.print(",L:");
  Serial.println(light);  // Lichtwert

  delay(10000);  // Alle 10 Sekunden neue Messwerte senden
}
