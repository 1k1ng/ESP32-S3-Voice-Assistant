## ESP32 S3 Voice Assistant with ESPHome

This repository contains a complete ESPHome setup for a compact voice assistant based on an ESP32 S3 DevKitC. It features an I2S amplifier for audio output, an I2S microphone for voice input, a WS2812 LED ring as a status indicator, and an SSD1306 OLED display for simple status expressions in a Pwnagotchi-inspired style.

## Features

1. Voice assistant integration in Home Assistant via ESPHome Voice Assistant
2. On-device Wake Word detection on the ESP32 via Micro Wake Word
3. Switchable Wake Word via a select entity in Home Assistant
4. Touch button control for toggling light and muting the microphone
5. LED ring status indicator for Ready, Listening, Thinking, Speaking, Mute, and Error
6. OLED display showing facial expressions based on the current state
7. Media player playback via the ESPHome Speaker Media Player
8. Automatic suspension of Wake Word and Voice Assistant during music playback

## Hardware

Components used in the shown setup:

1. ESP32 S3 DevKitC
2. I2S Class D Amplifier Breakout (e.g., MAX98357A)
3. I2S Microphone Module
4. WS2812 LED Ring with 45 LEDs
5. SSD1306 OLED Display 128x32 via I2C
6. Touch Button or push button input

The circuit diagram is included in the repository as a PNG file and shows the wiring matching the pins configured in the YAML.

## Pinout

WS2812 Data In: GPIO15  
Touch Button: GPIO17  

I2S BCLK: GPIO7  
I2S LRCLK: GPIO6  
Speaker Data Out: GPIO5  
Mic Data In: GPIO4  

I2C SDA: GPIO10  
I2C SCL: GPIO11  

Amplifier Power: 5V and GND  
OLED Power: 3V3 and GND  

Note: The microphone and speaker currently share the I2S clock lines. An optional plan for a future second I2S bus dedicated to the microphone is already commented out in the YAML file.

## Operation

Touch Button Actions:

1. Short click:  
   Toggle the LED ring light.

2. Double click:  
   Toggle microphone mute.

3. Long press:  
   Trigger the Voice Assistant (if not muted).

Releasing the button will stop an ongoing Voice Assistant session.

Wake Word Selection:

A select entity appears in Home Assistant with two options:

1. Jarvis
2. Nabu

Changing the selection dynamically enables and disables the respective models.

## Display & Indicator Behavior

OLED Display:

The display shows a centered face depending on the current state:

1. Mute active:  
   Angry look

2. Voice Assistant running:  
   Shows a different face during playback compared to when idle

3. Wake Word engine running:  
   Sleeping face

4. Idle:  
   Neutral face

LED Ring:

By default, the LED ring is turned off when idle. It only activates during specific states, such as Listening, Thinking, Speaking, or Mute. Additionally, there is a manual light mode that overrides the automatic turn-off behavior.

## Setup

Prerequisites:

1. Home Assistant with the ESPHome add-on installed
2. Wi-Fi credentials
3. API key and OTA password stored in secrets

Steps:

1. Create the YAML configuration in ESPHome and flash it to the device.
2. Fill out your `secrets.yaml` file with:  
   `wifi_ssid`  
   `wifi_password`  
   `api_key`  
   `ota_password`  
   `ap_password`  
3. Start the device and adopt it in Home Assistant.
4. Configure the Voice Assistant pipeline in Home Assistant if you haven't already.

## Important Audio Notes

1. The sample rate is set to 44100 Hz to match the media pipeline.
2. PSRAM is enabled to ensure stable buffer management.
3. During media playback, the Wake Word and Voice Assistant are paused, and the microphone is muted.
4. Once playback ends, the microphone is unmuted, and the Wake Word engine restarts.

## Troubleshooting

Wake Word is unreliable:

1. Check if the microphone is actually unmuted.
2. Ensure a stable 3V3 power supply and keep wire lengths short.
3. The Gain Factor in Micro Wake Word can be adjusted.
4. Make sure no media playback is running simultaneously.

No audio output:

1. Verify the I2S pins, especially BCLK, LRCLK, and DOUT.
2. Check the 5V and GND power supply to the amplifier.
3. If your amplifier has a Shutdown (SD) pin, you can optionally enable it in the YAML.

OLED remains blank:

1. Keep the I2C scan active and check the logs.
2. Address `0x3C` is standard, though some displays use `0x3D`.
3. Swap SDA and SCL lines if necessary.

## Recommended Project Structure

Optional, if you want to keep your repository organized:

1. `config` folder containing the ESPHome YAML
2. `docs` folder containing the circuit diagram and photos
3. `fonts` folder for local font files

## License

Choose your preferred license here (e.g., MIT) and add the LICENSE file to the repository.

## Acknowledgments

Thanks to the ESPHome and Home Assistant communities for providing the Voice Assistant, Micro Wake Word components, and audio framework.
