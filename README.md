## ESP32 S3 Voice Assistant with ESPHome

A local voice assistant for Home Assistant, built on an ESP32-S3 and ESPHome. It features on-device wake word detection, a speaker for answers and media playback, a status LED and an OLED display with simple text faces, a clock and timer countdowns.

There are two hardware approaches:

| | Waveshare board version | DIY circuit version |
|---|---|---|
| Hardware | Waveshare ESP32-S3 board with onboard audio codec, microphones, amplifier and sensors | ESP32-S3 DevKitC-1 plus separate breakout modules |
| Wiring | Almost none: only the OLED and the status LED are connected to the header | Everything has to be wired by hand (see [Wiring](#diy-circuit-version-wiring)) |
| Extras | Temperature/humidity sensor, real-time clock, 128x64 OLED | 128x32 OLED |
| Config file | `waveshare board version/esphome.yaml` | `esphome-diy-circuit.yaml` |

Both approaches are available in two case sizes (`big version` and `small version`). The ESPHome configuration is identical for both sizes; only the 3D-printed case differs.

## Repository Structure

```
big version/
├── esphome-diy-circuit.yaml              # ESPHome config for the DIY circuit
└── waveshare board version/
    ├── esphome.yaml                      # ESPHome config for the Waveshare board
    ├── VoiceAssistantProject.blend       # Blender source of the case
    └── VoiceAssistant_STL/               # Printable case parts
small version/
├── esphome-diy-circuit.yaml
└── waveshare board version/
    ├── esphome.yaml
    ├── smallAssistant_v2.blend
    └── small_assistant_STL/
esp32_s3_va.yaml                          # Legacy config (first prototype, see below)
```

## Features

1. Voice assistant integration in Home Assistant via the ESPHome Voice Assistant
2. On-device wake word detection via Micro Wake Word ("Hey Jarvis" and "Okay Nabu")
3. Media player for music, announcements and TTS answers
4. Timers: countdown on the OLED, alarm tone and flashing red LED when finished
5. Status LED for Listening, Thinking, Speaking, Mute and Error
6. Short beeps when the assistant starts and stops listening
7. OLED display with faces depending on the current state, otherwise a clock
8. Wake word and microphone are paused automatically during media playback

Additional features of the Waveshare board version:

1. Onboard SHTC3 temperature and humidity sensor with an adjustable temperature offset (compensates for heat from the ESP32 and amplifier; humidity is recalculated accordingly)
2. Onboard PCF85063 real-time clock, so the time is kept without Wi-Fi or Home Assistant
3. Idle screen with clock, room temperature and humidity
4. OLED burn-in protection (display is blanked for 1 second every 3 seconds)

## Home Assistant Entities

| Entity | Description |
|---|---|
| ESP32 Speaker | Media player |
| Mute | Mutes the microphone and stops the wake word |
| Night Mode | Turns the display off |
| Display Power | Turns the display on or off |
| Audio Output | `DAC and Amp` enables the amplifier, `DAC only` keeps it switched off |
| Status LED | The RGB status LED |
| Temperature / Humidity | Corrected room values (Waveshare only) |
| Temperature Offset | Offset in Kelvin subtracted from the raw temperature (Waveshare only) |
| Temperature (raw) / Humidity (raw) | Raw sensor values, disabled by default (Waveshare only) |

## Status Indicators

Status LED:

| State | LED |
|---|---|
| Idle | Off |
| Listening | Blue |
| Thinking | Orange |
| Speaking | Green, pulsing |
| Mute | Red |
| Error / timer finished | Red, fast pulsing |

OLED display (in order of priority):

1. Night Mode or Display Power off: blank
2. Mute active: `(-_-)`
3. Voice Assistant running: `(o_o)`, or `(^_^)` during media playback
4. Timer running: countdown
5. Idle: clock (Waveshare: clock, temperature and humidity)

## Waveshare Board Version

The Waveshare board already contains the ES8311 audio DAC, the ES7210 microphone ADC with onboard microphones, the amplifier, the SHTC3 sensor, the QMI8658 IMU and the PCF85063 RTC. You only need to connect a speaker, the OLED and the status LED.

| Component | Pin |
|---|---|
| SSD1306 OLED 128x64 SDA | GPIO45 |
| SSD1306 OLED 128x64 SCL | GPIO46 |
| WS2812 status LED data | GPIO44 |
| OLED / LED power | 3V3 / 5V and GND |

Notes:

1. Logging runs over the native USB port, because GPIO43/44 (default UART0) are used by the board.
2. Check that GPIO44 is accessible on your board's header before wiring the LED.
3. The QMI8658 IMU is prepared in the YAML but commented out, because the only available component targets the Arduino framework.

## DIY Circuit Version: Wiring

Components:

1. ESP32-S3 DevKitC-1 (with PSRAM recommended)
2. I2S Class D amplifier breakout, e.g. MAX98357A, plus a speaker
3. I2S MEMS microphone module with ICS-43434
4. SSD1306 OLED display 128x32 via I2C
5. Status LED: the onboard RGB LED of the DevKitC-1 (GPIO48) is used

| Module | Module pin | ESP32-S3 pin |
|---|---|---|
| Amplifier (MAX98357A) | BCLK | GPIO7 |
| | LRC | GPIO6 |
| | DIN | GPIO5 |
| | SD (shutdown / enable) | GPIO18 |
| | VIN / GND | 5V / GND |
| Microphone (ICS-43434) | BCLK / SCK | GPIO1 |
| | LRCL / WS | GPIO2 |
| | DOUT / SD | GPIO4 |
| | SEL / L/R | GND (left channel) |
| | VDD / GND | 3V3 / GND |
| OLED (SSD1306) | SDA | GPIO10 |
| | SCL | GPIO11 |
| | VCC / GND | 3V3 / GND |

Notes:

1. Microphone and speaker use two separate I2S buses.
2. Some DevKitC-1 revisions have the onboard RGB LED on GPIO38 instead of GPIO48. Adjust `led_pin` if the LED stays dark.
3. All pins are defined as substitutions at the top of the YAML and can be changed there.

## Setup

Prerequisites:

1. Home Assistant with the ESPHome add-on (or ESPHome dashboard)
2. A Voice Assistant pipeline configured in Home Assistant

Steps:

1. Copy the YAML of your chosen version into ESPHome.
2. Adjust `device_name` and `friendly_name` at the top of the file if you like.
3. Place the font file `DejaVuSans.ttf` in a `fonts` folder inside your ESPHome config directory (e.g. `/config/esphome/fonts/DejaVuSans.ttf`).
4. Add the following entries to your `secrets.yaml`:

   ```yaml
   wifi_ssid: "YourWifi"
   wifi_password: "YourWifiPassword"
   api_key: "32-byte base64 key"          # generate one in the ESPHome dashboard
   ota_password: "YourOtaPassword"
   ap_password: "FallbackHotspotPassword"

   # Static IP configuration
   wifi_gateway: 192.168.1.1
   wifi_subnet: 255.255.255.0
   diy_static_ip: 192.168.1.50            # DIY circuit version
   waveshare_static_ip: 192.168.1.51      # Waveshare board version
   legacy_static_ip: 192.168.1.52         # legacy config only
   ```

   If you prefer DHCP, remove the `manual_ip:` block from the YAML instead.

5. Flash the device, then adopt it in Home Assistant.

## Audio Notes

1. DIY version: the speaker runs at 44100 Hz, the microphone at 16000 Hz on its own I2S bus.
2. Waveshare version: microphone and speaker share the codec's clock lines. ESPHome cannot run both on one bus, so two I2S buses use the same pins, with the microphone as clock master. Everything runs at 16000 Hz.
3. PSRAM is enabled for stable buffer handling.
4. During media playback the wake word and Voice Assistant are paused and the microphone is muted; afterwards everything restarts automatically.

## Troubleshooting

Wake word is unreliable:

1. Check that Mute is off.
2. DIY: ensure a stable 3V3 supply and keep the microphone wires short.
3. Adjust `gain_factor` in `micro_wake_word`.
4. Make sure no media playback is running at the same time.

No audio output:

1. Check that Audio Output is set to `DAC and Amp`.
2. DIY: verify BCLK, LRC, DIN and the SD pin of the amplifier, and its 5V supply.

OLED stays blank:

1. Check that Night Mode is off and Display Power is on.
2. Keep the I2C scan enabled and look for the display address in the logs.
3. Address `0x3C` is standard; some displays use `0x3D`.
4. Swap SDA and SCL if necessary.

## Legacy Configuration

`esp32_s3_va.yaml` is the configuration of the first prototype (ESP32-S3 DevKitC, 45-LED WS2812 ring, touch button, 128x32 OLED, shared I2S bus). It is kept for reference and is no longer actively developed.

## License

This project is licensed under the [Creative Commons Attribution-NonCommercial 4.0 International License](LICENSE) (CC BY-NC 4.0).

## Acknowledgments

Thanks to the ESPHome and Home Assistant communities for the Voice Assistant, Micro Wake Word and audio components.
