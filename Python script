import serial
import time
import paho.mqtt.client as mqtt
import json

# MQTT-Einstellungen
MQTT_BROKER = "localhost"  # Home Assistant MQTT-Broker (meistens localhost auf dem Pi)
MQTT_PORT = 1883
MQTT_TOPIC_PREFIX = "home/arduino"
MQTT_DEVICE_NAME = "arduino_device"

# Serielle Verbindung zum Arduino
SERIAL_PORT = '/dev/ttyACM0'  # Der Port, an dem dein Arduino angeschlossen ist
BAUD_RATE = 9600

# Initialisiere serielle Kommunikation
ser = serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=1)

# MQTT-Client initialisieren
client = mqtt.Client()
client.connect(MQTT_BROKER, MQTT_PORT, 60)

def send_autodiscovery(sensor_type, sensor_name, unit_of_measurement):
    """Erstelle und sende die Autodiscovery-Nachricht für Home Assistant."""
    topic = f"homeassistant/sensor/{MQTT_DEVICE_NAME}/{sensor_type}/config"
    
    # Erstelle das Autodiscovery-Datenformat
    discovery_message = {
        "name": sensor_name,
        "state_topic": f"{MQTT_TOPIC_PREFIX}/{sensor_type}",
        "unit_of_measurement": unit_of_measurement,
        "device_class": sensor_type,
        "device": {
            "identifiers": [MQTT_DEVICE_NAME],
            "name": "Arduino Sensor",
            "model": "Arduino Uno",
            "manufacturer": "Arduino"
        }
    }
    
    # Sende die Autodiscovery-Nachricht
    client.publish(topic, json.dumps(discovery_message), retain=True)

def read_from_arduino():
    """Lese die Daten vom Arduino über die serielle Schnittstelle"""
    line = ser.readline().decode('utf-8').strip()  # Daten vom Arduino lesen
    return line

def ldr_to_lux(ldr_value):
    """Umrechnung von LDR-Wert auf Lux (Max. 100.000 Lux)"""
    lux = (ldr_value / 1023.0)
    return round(lux, 2)  # Gibt den Lux-Wert zurück

def publish_to_mqtt(data):
    """Veröffentliche Daten über MQTT und sende Autodiscovery-Nachrichten"""
    try:
        # Aufteilen der Daten in Temperatur, Feuchtigkeit und Licht
        parts = data.split(",")
        temp = float(parts[0].split(":")[1])
        humid = float(parts[1].split(":")[1])
        light = int(parts[2].split(":")[1])

        # Umrechnung von LDR-Wert in Lux
        lux_value = ldr_to_lux(light)

        # Autodiscovery für jeden Sensor
        send_autodiscovery("temperature", "Arduino Temperatur", "°C")
        send_autodiscovery("humidity", "Arduino Luftfeuchtigkeit", "%")
        send_autodiscovery("illuminance", "Arduino Licht", "lx")  # "lx" für Lux

        # Veröffentliche die Daten auf den jeweiligen MQTT-Themen
        client.publish(f"{MQTT_TOPIC_PREFIX}/temperature", temp)
        client.publish(f"{MQTT_TOPIC_PREFIX}/humidity", humid)
        client.publish(f"{MQTT_TOPIC_PREFIX}/illuminance", lux_value)  # Jetzt im Bereich 0 bis 100.000 Lux
        print(f"Published: Temp={temp}°C, Humid={humid}%, Light={lux_value} lx")
    
    except Exception as e:
        print(f"Error: {e}")

def main():
    while True:
        data = read_from_arduino()  # Daten vom Arduino lesen
        if data:
            publish_to_mqtt(data)  # Daten an den MQTT-Broker senden
        time.sleep(3)  # 3 Sekunden warten

if __name__ == "__main__":
    main()
