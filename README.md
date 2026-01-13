<p align="center">
  <img src="https://www.arduino.cc/en/uploads/Main/arduino-logo-small.png" alt="Arduino Logo" width="200"/>
</p>

# ESP32 Weather Station with BME280

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Arduino](https://img.shields.io/badge/Arduino-00979D?style=for-the-badge&logo=arduino&logoColor=white)
![ESP32](https://img.shields.io/badge/ESP32-E7352C?style=for-the-badge&logo=espressif&logoColor=white)

A simple but powerful weather station based on an ESP32 and a BME280 sensor for measuring temperature, humidity, and atmospheric pressure. The data is displayed on a crisp SSD1306 OLED display.

## ✨ Features

-   🌡️ **Temperature measurement** in Celsius
-   💧 **Humidity measurement** in %RH
-   🎈 **Pressure measurement** in hPa
-   📺 **OLED display** for data visualization
-   🤖 **FreeRTOS-based** for stable and parallel execution

## 🛠️ Hardware

| Component       | Description                                |
| --------------- | ------------------------------------------ |
| **ESP32**       | Microcontroller with integrated WiFi/BT    |
| **BME280**      | Sensor for temperature, humidity, pressure |
| **SSD1306**     | 128x64 I2C OLED Display                    |
| **Breadboard**  | For easy assembly                          |
| **Cables**      | Dupont cables for connections              |

## ⚙️ Software & Installation

### Required Libraries

Make sure the following libraries are installed in your Arduino IDE:

-   `Adafruit GFX Library`
-   `Adafruit SSD1306`
-   `Adafruit BME280 Library`

### Setup

1.  **Wiring:** Connect the components as follows:
    -   **BME280 & SSD1306 to ESP32:**
        -   `SDA` -> `GPIO 21`
        -   `SCL` -> `GPIO 22`
        -   `VCC` -> `3.3V`
        -   `GND` -> `GND`
2.  **Arduino IDE:**
    -   Open `Wetterstation/Wetterstation.ino`.
    -   Select the board "ESP32 Dev Module".
    -   Select the correct COM port.
3.  **Upload:** Compile and upload the sketch to the ESP32.

## 📁 Code Structure

The project is divided into multiple files for better organization:

```
Wetterstation/
├── Wetterstation.ino   # Main file with setup() and loop()
├── config.h            # Configuration (pins, addresses)
├── globals.h           # Global variables and objects
└── tasks.h             # RTOS tasks for measurement and display
```

## 📜 License

This project is licensed under the **MIT License**. See the [LICENSE](LICENSE) file for more details.
