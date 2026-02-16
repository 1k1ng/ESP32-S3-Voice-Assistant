## ESP32 S3 Voice Assistant mit ESPHome

Dieses Repository enthält ein komplettes ESPHome Setup für einen kompakten Sprachassistenten auf Basis eines ESP32 S3 DevKitC. Mit dabei sind ein I2S Verstärker für Audioausgabe, ein I2S Mikrofon für Spracheingabe, ein WS2812 LED Ring als Statusanzeige und ein SSD1306 OLED Display für einfache Zustände im Pwnagotchi Stil.

## Funktionen

1. Sprachassistent in Home Assistant über ESPHome Voice Assistant
2. Wake Word Erkennung direkt auf dem ESP32 über Micro Wake Word
3. Umschaltbares Wake Word über ein Select in Home Assistant
4. Touch Button Bedienung für Licht und Mikrofon Stummschaltung
5. LED Ring Statusanzeige für Ready, Listening, Thinking, Speaking, Mute, Error
6. OLED Display mit Gesichtsausdrücken je nach Zustand
7. Media Player Wiedergabe über den ESPHome Speaker Media Player
8. Automatisches Stoppen von Wake Word und Voice Assistant während Musikwiedergabe

## Hardware

Verwendete Bauteile im gezeigten Aufbau

1. ESP32 S3 DevKitC
2. I2S Class D Verstärker Breakout, zum Beispiel MAX98357A
3. I2S Mikrofon Modul
4. WS2812 LED Ring mit 45 LEDs
5. SSD1306 OLED Display 128x32 über I2C
6. Touch Button oder Taster Eingang

Das Schaltbild als PNG ist im Repository enthalten und zeigt die Verdrahtung passend zu den Pins in der YAML.

## Pinbelegung

WS2812 Data In: GPIO15
Touch Button: GPIO17

I2S BCLK: GPIO7
I2S LRCLK: GPIO6
Speaker Data Out: GPIO5
Mic Data In: GPIO4

I2C SDA: GPIO10
I2C SCL: GPIO11

Versorgung Verstärker: 5V und GND
Versorgung OLED: 3V3 und GND

Hinweis: Mikrofon und Lautsprecher teilen sich aktuell die I2S Clock Leitungen. In der YAML ist bereits ein optionaler Plan für einen späteren zweiten I2S Bus für das Mikrofon kommentiert.

## Bedienung

Touch Button Aktionen

1. Kurzer Klick
   LED Ring umschalten

2. Doppelklick
   Mikrofon Stummschaltung umschalten

3. Langer Druck
   Voice Assistant starten, sofern nicht stumm

Beim Loslassen wird ein laufender Voice Assistant wieder gestoppt.

Wake Word Auswahl

In Home Assistant erscheint ein Select mit zwei Optionen

1. Jarvis
2. Nabu

Beim Wechsel werden die Modelle entsprechend aktiviert und deaktiviert.

## Anzeigeverhalten

OLED Display

Das Display zeigt je nach Zustand ein zentriertes Gesicht

1. Mute aktiv
   Böser Blick

2. Voice Assistant läuft
   Beim Abspielen anderes Gesicht als ohne Wiedergabe

3. Wake Word Engine läuft
   Schlafendes Gesicht

4. Idle
   Neutrales Gesicht

LED Ring

Standardmäßig ist der LED Ring im Ruhezustand aus. Nur bei bestimmten Zuständen wird er aktiv, zum Beispiel Listening, Thinking, Speaking oder Mute. Zusätzlich gibt es einen manuellen Licht Modus, der das automatische Aus Verhalten übersteuert.

## Einrichtung

Voraussetzungen

1. Home Assistant mit ESPHome Addon
2. WLAN Zugangsdaten
3. API Key und OTA Passwort in secrets

Schritte

1. YAML in ESPHome anlegen und ins Gerät flashen
2. secrets Datei befüllen
   wifi_ssid
   wifi_password
   api_key
   ota_password
   ap_password
3. Gerät starten und in Home Assistant übernehmen
4. In Home Assistant den Voice Assistant Dienst konfigurieren, falls noch nicht vorhanden

## Wichtige Hinweise zu Audio

1. Sample Rate ist auf 44100 gesetzt, passend zur Media Pipeline
2. PSRAM ist aktiv, um Puffer stabil zu halten
3. Während Media Playback werden Wake Word und Voice Assistant gestoppt und das Mikrofon gemutet
4. Nach Ende der Wiedergabe wird das Mikrofon wieder aktiviert und Wake Word erneut gestartet

## Fehlerbehebung

Wake Word reagiert nicht zuverlässig

1. Prüfe, ob das Mikrofon wirklich ungemutet ist
2. Achte auf ausreichende 3V3 Versorgung und kurze Leitungen
3. Gain Factor in Micro Wake Word kann angepasst werden
4. Stelle sicher, dass keine gleichzeitige Media Wiedergabe läuft

Keine Audioausgabe

1. I2S Pins prüfen, besonders BCLK, LRCLK, DOUT
2. Verstärker Versorgung 5V und GND prüfen
3. Falls dein Verstärker einen Shutdown Pin hat, optional in der YAML aktivieren

OLED bleibt leer

1. I2C Scan aktiv lassen und Log prüfen
2. Adresse 0x3C ist Standard, manche Displays nutzen 0x3D
3. SDA und SCL tauschen falls nötig

## Projektstruktur Empfehlung

Optional, falls du es im Repository sauber trennen willst

1. config Ordner mit der ESPHome YAML
2. docs Ordner mit Schaltbild und Fotos
3. fonts Ordner für lokale Schriftarten

## Lizenz

Wähle hier deine gewünschte Lizenz, zum Beispiel MIT, und füge die Lizenzdatei ins Repository ein.

## Danksagung

ESPHome und Home Assistant Community für Voice Assistant, Micro Wake Word und die Audio Komponenten.
