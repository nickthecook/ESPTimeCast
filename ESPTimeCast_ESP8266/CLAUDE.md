# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ESPTimeCast is an ESP8266-based WiFi LED matrix clock and weather station that displays time, date, weather information, and countdown timers on a MAX7219 LED matrix display. This is the ESP8266 variant of the project.

## Build and Development Commands

Since this is an Arduino IDE project without platformio.ini, compilation is handled through the Arduino IDE:

1. **Compile**: Use Arduino IDE's "Verify" button (checkmark icon) to compile the sketch
2. **Upload**: Use Arduino IDE's "Upload" button (right arrow icon) to flash the ESP8266
3. **Upload filesystem**: Use the LittleFS uploader plugin to upload the `/data` folder contents

**Required Libraries** (install via Arduino IDE Library Manager):
- ArduinoJson by Benoit Blanchon
- MD_Parola by majicDesigns (also installs MD_MAX72xx dependency)
- ESPAsyncTCP by ESP32Async
- ESPAsyncWebServer by ESP32Async

**Board Configuration**:
- Board: "Wemos D1 Mini" (or equivalent ESP8266 variant)
- Flash Size: "4MB FS:2MB OTA:~1019KB"
- ESP8266 Board Package URL: `http://arduino.esp8266.com/stable/package_esp8266com_index.json`

## Code Architecture

### Core Components

**Main File**: `ESPTimeCast_ESP8266.ino` - Single-file Arduino sketch containing all functionality

**Configuration System**:
- `data/config.json` - Persistent configuration stored in LittleFS
- Web interface for configuration via AsyncWebServer
- Backup/restore functionality for settings

**Display Management**:
- MD_Parola library for MAX7219 LED matrix control
- Custom font defined in `mfactoryfont.h`
- Four display modes: Clock, Weather, Weather Description, Countdown
- State machine for display mode switching with configurable durations

**Localization Support**:
- `tz_lookup.h` - IANA to POSIX timezone mappings for 80+ timezones
- `days_lookup.h` - Day-of-week translations for 25+ languages
- `months_lookup.h` - Month name translations for 25+ languages
- Text format uses `&` separators for LED display character spacing

### Key Functional Areas

**Network Management**:
- WiFi connection with fallback to AP mode
- AsyncWebServer for configuration interface
- DNSServer for captive portal in AP mode
- Non-blocking HTTP weather requests

**Time Synchronization**:
- NTP client with state machine for robust sync handling
- Configurable primary/secondary NTP servers
- Timezone handling with DST support via POSIX strings
- Retry logic with timeouts

**Weather Integration**:
- OpenWeatherMap API integration
- 5-minute fetch intervals
- Temperature, humidity, and description display
- Error handling for API failures

**Display Features**:
- Brightness control (0-15) with optional dimming schedule
- Display flip (180-degree rotation)
- Scrolling text for long descriptions
- Blinking colon separator
- Status icons for various states

**Countdown System**:
- Unix timestamp-based target setting
- Two modes: scrolling and dramatic countdown
- "TIMES UP" message with flashing display
- Automatic mode switching when countdown expires

### Hardware Configuration

**GPIO Pin Mapping** (Wemos D1 Mini to MAX7219):
- CLK_PIN (12) → D6 → CLK
- DATA_PIN (15) → D8 → DIN
- CS_PIN (13) → D7 → CS
- Power: 5V rail (not 3.3V) to prevent regulator overload

**Display Setup**:
- HARDWARE_TYPE: MD_MAX72XX::FC16_HW
- MAX_DEVICES: 4 (8x32 pixel display)
- Custom scroll speeds for different content types

### Development Notes

**Configuration Management**:
- All settings stored in LittleFS as JSON
- Atomic save operations with backup
- Factory reset functionality
- Settings validation and migration

**Error Handling**:
- NTP sync failure recovery
- WiFi reconnection logic
- Weather API error handling
- Display fallback modes

**Performance Considerations**:
- Non-blocking weather fetches
- Efficient string handling for memory-constrained ESP8266
- Minimal JSON parsing for configuration
- Optimized display update cycles

**Web Interface**:
- Single-page application served from LittleFS
- RESTful endpoints for configuration
- Location-based timezone detection
- Advanced settings panel

## Important Implementation Details

- Configuration changes require device reboot to take effect
- Weather descriptions use language-specific character mapping
- Timezone selection automatically handles DST transitions
- Display brightness can be scheduled for automatic dimming
- Countdown timestamps are Unix epoch-based
- All text uses custom character encoding for LED matrix compatibility