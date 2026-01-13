# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ESP32-based weather station using a BME280 sensor to measure temperature, humidity, and pressure, displaying the data on an SSD1306 OLED display. The project uses FreeRTOS for concurrent task execution.

## Build and Upload

This is an Arduino project for ESP32. To compile and upload:

1. Open `Wetterstation/Wetterstation.ino` in Arduino IDE
2. Select **Board**: "ESP32 Dev Module"
3. Select the correct **COM Port**
4. Click Upload to compile and flash to the ESP32

**Serial Monitor**: Set to 115200 baud to view debug output from the measurement task.

## Required Libraries

Install these libraries via Arduino IDE Library Manager:

- `Adafruit GFX Library`
- `Adafruit SSD1306`
- `Adafruit BME280 Library`
- `Wire` (included with Arduino core)

## Hardware Configuration

**I2C Bus:**
- SDA: GPIO 21
- SCL: GPIO 22
- BME280 Address: `0x76`
- SSD1306 Address: `0x3D`

All I2C devices share the same bus and are synchronized via mutex.

## Architecture

### FreeRTOS Task-Based Design

The project uses FreeRTOS for concurrent execution with two independent tasks:

1. **`taskMeasure`** (tasks.h:9-29)
   - Runs every 2 seconds
   - Reads temperature, humidity, and pressure from BME280
   - Updates shared `currentData` struct
   - Outputs measurements to serial console

2. **`taskDisplay`** (tasks.h:32-66)
   - Runs every 1 second
   - Renders weather data on SSD1306 OLED display
   - Formats: temperature (1 decimal), humidity (integer), pressure (integer)

Both tasks are pinned to **Core 1** with priority 1 and 4096 bytes of stack.

### I2C Bus Synchronization

**Critical**: Both the BME280 sensor and SSD1306 display use the I2C bus. Access is synchronized using a FreeRTOS mutex (`i2cMutex`):

```cpp
if (xSemaphoreTake(i2cMutex, portMAX_DELAY) == pdTRUE) {
    // Perform I2C operations
    xSemaphoreGive(i2cMutex);
}
```

The mutex is created in `setup()` before any I2C initialization. Any code accessing I2C devices must acquire this mutex first.

### File Structure

- **`Wetterstation.ino`**: Main file with `setup()` and `loop()`
  - Initializes hardware (Wire, display, sensor)
  - Creates mutex and spawns tasks
  - `loop()` is intentionally empty - all logic runs in tasks

- **`config.h`**: Configuration constants
  - Display dimensions, I2C addresses, sea level pressure

- **`globals.h`**: Global objects and shared data
  - Adafruit sensor/display objects
  - `i2cMutex` semaphore handle
  - `WeatherData` struct for inter-task communication

- **`tasks.h`**: FreeRTOS task definitions
  - Complete implementations of both tasks

### Shared Data

The `WeatherData` struct (globals.h:18-22) is used to pass measurements from `taskMeasure` to `taskDisplay`:

```cpp
struct WeatherData {
  float temp;  // Celsius
  float hum;   // % Relative Humidity
  float pres;  // hPa
};
```

Access to `currentData` is implicitly synchronized through the I2C mutex since both tasks hold the mutex when reading/writing.

## Development Guidelines

### Adding New I2C Devices

When adding new I2C sensors or displays:

1. Define I2C address in `config.h`
2. Create object in `globals.h`
3. Initialize in `setup()` after Wire.begin()
4. **Always** acquire `i2cMutex` before I2C operations
5. Release mutex immediately after operations complete

### Modifying Task Timing

- Measurement interval: Change delay in `taskMeasure` (line 27)
- Display refresh rate: Change delay in `taskDisplay` (line 64)
- Use `vTaskDelay(pdMS_TO_TICKS(milliseconds))` for non-blocking delays

### Task Communication

If adding new tasks that need to share data:
- Define shared data structures in `globals.h`
- Consider using FreeRTOS queues or additional mutexes for synchronization
- Document which mutex protects which data

### Core Pinning

Both tasks currently run on Core 1. Core 0 is reserved for WiFi/BLE if needed. To change core assignment, modify the last parameter in `xTaskCreatePinnedToCore()` (0 or 1).

## Future Extensions (from FSD)

Potential improvements documented in `Wetterstation_fsd.md`:
- WiFi connectivity for data transmission
- Data logging to SD card or flash
- Web server for local network access
- Sensor calibration
- Real-time clock integration
