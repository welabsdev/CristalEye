# CristalEye for Robot

![Version](https://img.shields.io/badge/version-0.3.0-blue.svg)
![Platform](https://img.shields.io/badge/platform-Arduino/EspressifSystems-0078D6.svg)
![License](https://img.shields.io/badge/license-GPL--3.0-green.svg)



### C++ library for animated and expressive robot eyes on graphical displays.

WELABSDEV CristalEye is a lightweight, modern and object-oriented C++ library designed to create animated, expressive and autonomous robot eyes on OLED and graphical displays.

The library is built around the drawing primitives provided by [Adafruit GFX](https://github.com/adafruit/Adafruit-GFX-Library) and provides smooth mathematical interpolation (tweening), procedural behaviors and non-blocking animations without using `delay()`.

The main goal is to provide a simple API for adding visual personality to robotics projects while keeping the rendering engine lightweight enough for microcontrollers.

---

## Features

- Modern C++ API
- Header-only architecture
- Template-based display abstraction
- Adafruit GFX compatibility
- Non-blocking animation engine
- `millis()`-based timing
- Configurable frame rate
- Smooth position interpolation
- Automatic blinking
- Autonomous idle behavior
- Multiple eye positions
- Multiple emotional states
- Curiosity behavior
- Cyclops mode
- Sweat animation
- Confusion animation
- Laugh animation
- Manual blink control
- Optimized integer types for embedded systems
- Strongly typed `enum class` API
- Designed for ESP32 and similar microcontrollers

---

## Architecture

CristalEye separates the animation engine from the display driver through C++ templates.

```cpp
CristalEye<DisplayDriver> eyes(display);
```

This allows the same animation system to be used with different display implementations without modifying the core library.

Conceptually:

```text
Application
    |
    v
CristalEye
    |
    +---- Animation Engine
    |
    +---- Behavior System
    |
    +---- Tweening
    |
    +---- Rendering
    |
    v
Adafruit GFX compatible display
```

---

## Requirements

### Hardware

CristalEye is designed primarily for microcontrollers such as:

- ESP32
- ESP32-S2
- ESP32-S3
- ESP8266
- Arduino-compatible boards
- Other MCUs supported by the required display libraries

### Software

- Arduino IDE
- C++11 or newer
- Adafruit GFX Library
- Display-specific Adafruit library

For ESP32 projects, install the official ESP32 board package through the Arduino IDE Board Manager.

---

## Installation

### 1. Install ESP32 support

In Arduino IDE:

```text
File
  -> Preferences
```

Add the ESP32 package URL to:

```text
Additional Boards Manager URLs
```

Use:

```text
https://dl.espressif.com/dl/package_esp32_index.json
```

Then open:

```text
Tools
  -> Board
      -> Boards Manager
```

Search for:

```text
esp32
```

and install the ESP32 platform provided by Espressif.

---

### 2. Install Adafruit libraries

Open:

```text
Sketch
  -> Include Library
      -> Manage Libraries
```

Install:

```text
Adafruit GFX Library
```

Then install the library corresponding to your display.

For an SSD1306 OLED:

```text
Adafruit SSD1306
```

If Arduino IDE asks to install missing dependencies, select:

```text
Install All
```

---

### 3. Install CristalEye

CristalEye is currently designed as a header-only library.

Place:

```text
cristaleye.h
```

inside the same directory as your Arduino sketch.

Example:

```text
MeuRobo/
├── MeuRobo.ino
└── cristaleye.h
```

Then include it in your sketch:

```cpp
#include "cristaleye.h"
```

---

# Quick Start

The following example demonstrates CristalEye running on an ESP32 with an SSD1306 128x64 I2C OLED.

## Example

```cpp
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#include "cristaleye.h"

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1

Adafruit_SSD1306 display(
    SCREEN_WIDTH,
    SCREEN_HEIGHT,
    &Wire,
    OLED_RESET
);

CristalEye<Adafruit_SSD1306> robotEyes(display);

void setup()
{
    Serial.begin(115200);

    if (!display.begin(
        SSD1306_SWITCHCAPVCC,
        0x3C
    ))
    {
        Serial.println(F("Failed to initialize SSD1306"));

        for (;;);
    }

    robotEyes.iniciar(
        SCREEN_WIDTH,
        SCREEN_HEIGHT,
        50
    );

    robotEyes.setCoresDisplay(0, 1);

    robotEyes.setModoInativo(
        true,
        2,
        4
    );

    robotEyes.setPiscarAutomatico(
        true,
        3,
        2
    );

    robotEyes.setCuriosidade(true);

    robotEyes.setHumor(
        Humor::FELIZ
    );
}

void loop()
{
    robotEyes.atualizar();
}
```

The important part is that `atualizar()` must be called continuously.

```cpp
void loop()
{
    robotEyes.atualizar();
}
```

Do not use `delay()` inside the main animation loop.

---

# API Reference

## Lifecycle

### `iniciar()`

Initializes the CristalEye engine.

```cpp
iniciar(
    int16_t largura,
    int16_t altura,
    uint8_t fps
);
```

Example:

```cpp
robotEyes.iniciar(128, 64, 50);
```

Parameters:

| Parameter | Type | Description |
|---|---|---|
| `largura` | `int16_t` | Display width |
| `altura` | `int16_t` | Display height |
| `fps` | `uint8_t` | Target animation frame rate |

---

### `atualizar()`

Updates the animation engine and renders the current frame.

```cpp
robotEyes.atualizar();
```

This method should be called continuously from `loop()`.

Recommended:

```cpp
void loop()
{
    robotEyes.atualizar();

    // Other robot tasks
}
```

---

### `setFramerate()`

Changes the target animation frame rate.

```cpp
robotEyes.setFramerate(60);
```

Example:

```cpp
robotEyes.setFramerate(30);
```

Higher FPS can provide smoother animations but may increase CPU and display workload.

---

# Display Configuration

## `setCoresDisplay()`

Defines the background and drawing colors.

```cpp
robotEyes.setCoresDisplay(
    0,
    1
);
```

For monochrome SSD1306 displays:

```text
0 = Black
1 = White
```

Example:

```cpp
robotEyes.setCoresDisplay(0, 1);
```

---

# Emotions

CristalEye provides a strongly typed emotion system.

## `Humor`

Available values:

```cpp
Humor::PADRAO
Humor::CANSADO
Humor::IRRITADO
Humor::FELIZ
```

Example:

```cpp
robotEyes.setHumor(
    Humor::FELIZ
);
```

Change the expression at runtime:

```cpp
robotEyes.setHumor(
    Humor::CANSADO
);
```

---

# Eye Position

The eye direction is controlled through the `Posicao` enum class.

Available positions:

```cpp
Posicao::N
Posicao::NE
Posicao::L
Posicao::SE
Posicao::S
Posicao::SO
Posicao::O
Posicao::NO
Posicao::CENTRO
```

The positions represent:

| Position | Direction |
|---|---|
| `N` | North |
| `NE` | Northeast |
| `L` | East |
| `SE` | Southeast |
| `S` | South |
| `SO` | Southwest |
| `O` | West |
| `NO` | Northwest |
| `CENTRO` | Center |

Example:

```cpp
robotEyes.setPosicao(
    Posicao::NE
);
```

Return to center:

```cpp
robotEyes.setPosicao(
    Posicao::CENTRO
);
```

---

# Autonomous Behaviors

CristalEye can generate behaviors automatically without requiring application code to manually control every animation.

## Automatic Blinking

Enable automatic blinking:

```cpp
robotEyes.setPiscarAutomatico(
    true,
    3,
    2
);
```

Signature:

```cpp
setPiscarAutomatico(
    bool ativo,
    int16_t intervaloSegundos,
    int16_t variacaoSegundos
);
```

The system uses the configured interval and variation to produce less predictable and more organic blinking behavior.

Disable it:

```cpp
robotEyes.setPiscarAutomatico(
    false,
    0,
    0
);
```

---

## Idle Mode

Idle Mode allows the robot to periodically change its gaze automatically.

```cpp
robotEyes.setModoInativo(
    true,
    2,
    4
);
```

Signature:

```cpp
setModoInativo(
    bool ativo,
    int16_t intervaloSegundos,
    int16_t variacaoSegundos
);
```

This can be useful when the robot has no active interaction.

Example:

```cpp
robotEyes.setModoInativo(
    true,
    2,
    4
);
```

The robot can autonomously look in different directions while the rest of the application continues running.

---

## Curiosity

Enable curiosity behavior:

```cpp
robotEyes.setCuriosidade(true);
```

This modifies the eye behavior when looking toward the edges of the display, creating a more expressive visual response.

Disable it:

```cpp
robotEyes.setCuriosidade(false);
```

---

## Cyclops Mode

CristalEye can transition between two eyes and a single central eye.

```cpp
robotEyes.setCiclope(true);
```

Disable:

```cpp
robotEyes.setCiclope(false);
```

The transition is handled by the animation system rather than requiring an immediate visual switch.

---

## Sweat Animation

Enable the sweat effect:

```cpp
robotEyes.setSuor(true);
```

Disable:

```cpp
robotEyes.setSuor(false);
```

This provides a visual reaction suitable for expressive or anime-inspired robot characters.

---

# Instant Animations

CristalEye also provides direct animation commands.

## Blink

Force an immediate blink:

```cpp
robotEyes.piscar(
    true,
    true
);
```

The first parameter controls the left eye and the second controls the right eye.

Left eye only:

```cpp
robotEyes.piscar(
    true,
    false
);
```

Right eye only:

```cpp
robotEyes.piscar(
    false,
    true
);
```

---

## Confusion Animation

Starts a rapid horizontal shaking animation.

```cpp
robotEyes.animacaoConfuso();
```

Example:

```cpp
if (sensorError)
{
    robotEyes.animacaoConfuso();
}
```

---

## Laugh Animation

Starts a vertical oscillation animation.

```cpp
robotEyes.animacaoRiso();
```

Example:

```cpp
if (robotHappy)
{
    robotEyes.animacaoRiso();
}
```

---

# Complete Behavior Example

The following example demonstrates multiple behaviors working together.

```cpp
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#include "cristaleye.h"

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1

Adafruit_SSD1306 display(
    SCREEN_WIDTH,
    SCREEN_HEIGHT,
    &Wire,
    OLED_RESET
);

CristalEye<Adafruit_SSD1306> eyes(display);

void setup()
{
    Serial.begin(115200);

    if (!display.begin(
        SSD1306_SWITCHCAPVCC,
        0x3C
    ))
    {
        Serial.println(
            F("SSD1306 initialization failed")
        );

        while (true);
    }

    eyes.iniciar(
        SCREEN_WIDTH,
        SCREEN_HEIGHT,
        50
    );

    eyes.setCoresDisplay(0, 1);

    eyes.setHumor(
        Humor::PADRAO
    );

    eyes.setModoInativo(
        true,
        2,
        4
    );

    eyes.setPiscarAutomatico(
        true,
        3,
        2
    );

    eyes.setCuriosidade(true);

    eyes.setCiclope(false);

    eyes.setSuor(false);
}

void loop()
{
    eyes.atualizar();

    /*
     * Other robot tasks can execute here.
     *
     * readSensors();
     * updateMotors();
     * processBluetooth();
     * processWiFi();
     * runAI();
     */
}
```

---

# Non-Blocking Design

One of the main design principles of CristalEye is avoiding blocking operations.

A traditional animation could look like:

```cpp
moveEyes();

delay(100);

blink();

delay(200);

lookRight();

delay(500);
```

This approach prevents the microcontroller from performing other tasks during the animation.

CristalEye instead follows a cooperative update model:

```cpp
void loop()
{
    eyes.atualizar();

    readSensors();
    updateMotors();
    processCommunication();
    updateRobotLogic();
}
```

The animation engine determines whether the next frame needs to be rendered based on elapsed time.

This makes the library suitable for robots that need to perform multiple operations simultaneously.

---

# Performance

The animation engine is designed around `millis()` rather than blocking delays.

Conceptually:

```text
loop()
  |
  +-- atualizar()
  |      |
  |      +-- Check elapsed time
  |      |
  |      +-- Calculate animation progress
  |      |
  |      +-- Interpolate position
  |      |
  |      +-- Apply expression
  |      |
  |      +-- Render frame
  |
  +-- Sensors
  |
  +-- Motors
  |
  +-- Communication
  |
  +-- Application logic
```

The configured FPS determines how frequently the display is updated.

Example:

```cpp
eyes.setFramerate(50);
```

or:

```cpp
eyes.setFramerate(60);
```

---

# Type Safety

Instead of relying on numeric constants:

```cpp
setHumor(3);
setPosicao(1);
```

CristalEye uses strongly typed enumerations:

```cpp
setHumor(
    Humor::FELIZ
);

setPosicao(
    Posicao::NE
);
```

This improves readability and prevents accidental mixing of unrelated values.

---

# Memory Optimization

Embedded systems often have significantly less memory than desktop systems.

CristalEye therefore favors explicit-width integer types where appropriate:

```cpp
uint8_t
int16_t
```

This provides predictable storage requirements and helps reduce unnecessary memory usage.

The library is intended to remain lightweight enough for microcontroller-based robotics projects.

---

# Supported Displays

CristalEye is designed around the Adafruit GFX drawing API.

Depending on the implementation of the display driver, it can be used with displays such as:

| Display | Typical Driver |
|---|---|
| SSD1306 | `Adafruit_SSD1306` |
| SSD1327 | `Adafruit_SSD1327` |
| SH1106 | GFX-compatible implementations |
| ST7735 | `Adafruit_ST7735` |
| Other GFX displays | Depends on driver |

Example:

```cpp
CristalEye<Adafruit_SSD1306> eyes(display);
```

The display class is supplied as the template parameter.

---

# Project Structure

A minimal Arduino project can look like:

```text
MeuRobo/
│
├── MeuRobo.ino
│
└── cristaleye.h
```

For a larger project:

```text
MeuRobo/
│
├── MeuRobo.ino
│
├── cristaleye.h
│
├── sensors.h
│
├── motors.h
│
└── robot_config.h
```

---

# Recommended Main Loop

CristalEye works best when `atualizar()` is called continuously.

Recommended:

```cpp
void loop()
{
    eyes.atualizar();

    readSensors();
    updateMotors();
    processCommunication();
    updateRobot();
}
```

Avoid:

```cpp
void loop()
{
    eyes.atualizar();

    delay(100);
}
```

Large blocking delays reduce animation responsiveness and can interfere with other robot systems.

---

# API Summary

| Method | Description |
|---|---|
| `iniciar()` | Initializes the animation engine |
| `atualizar()` | Updates and renders the current frame |
| `setFramerate()` | Changes the target FPS |
| `setCoresDisplay()` | Configures display colors |
| `setHumor()` | Changes the eye expression |
| `setPosicao()` | Changes the eye direction |
| `setPiscarAutomatico()` | Enables automatic blinking |
| `setModoInativo()` | Enables autonomous idle behavior |
| `setCuriosidade()` | Enables curiosity behavior |
| `setCiclope()` | Enables cyclops mode |
| `setSuor()` | Enables sweat animation |
| `piscar()` | Forces an immediate blink |
| `animacaoConfuso()` | Starts confusion animation |
| `animacaoRiso()` | Starts laugh animation |

---

# Roadmap

The project is designed to evolve toward a more complete expressive animation framework for robotics.

Possible future features include:

- More emotional states
- Additional eye shapes
- Eyebrow rendering
- Advanced pupil behavior
- Eye tracking
- Sensor-driven expressions
- Audio-reactive expressions
- More procedural animations
- Configurable animation curves
- Custom animation sequences
- Additional display optimizations
- Support for more GFX-compatible drivers
- External animation configuration
- More autonomous behaviors

---

# Contributing

Contributions are welcome.

Before submitting a pull request:

1. Keep the API simple and consistent.
2. Avoid unnecessary dynamic memory allocation.
3. Avoid blocking operations.
4. Prefer fixed-width integer types when appropriate.
5. Maintain compatibility with embedded environments.
6. Keep platform-specific code isolated whenever possible.
7. Document new public APIs.

For larger changes, open an issue first to discuss the proposed architecture.

---

# License

Add your project's license information here.

Example:

```text
MIT License
```

If this project is distributed under the MIT License, include the complete `LICENSE` file in the repository.

---

# Author

**Wenderson Dias**

WELABSDEV

CristalEye was created as part of the WELABSDEV robotics and embedded systems ecosystem, with the goal of providing lightweight and expressive interfaces for maker and professional robotics projects.

---

# Acknowledgements

CristalEye is built around the graphics primitives provided by the Adafruit GFX ecosystem.

Special thanks to the open-source community and the developers maintaining the Arduino and Adafruit ecosystems.

---

# Final Example

A complete minimal implementation:

```cpp
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#include "cristaleye.h"

#define WIDTH 128
#define HEIGHT 64
#define RESET -1

Adafruit_SSD1306 display(
    WIDTH,
    HEIGHT,
    &Wire,
    RESET
);

CristalEye<Adafruit_SSD1306> eyes(display);

void setup()
{
    display.begin(
        SSD1306_SWITCHCAPVCC,
        0x3C
    );

    eyes.iniciar(
        WIDTH,
        HEIGHT,
        50
    );

    eyes.setCoresDisplay(0, 1);

    eyes.setHumor(
        Humor::FELIZ
    );

    eyes.setCuriosidade(true);

    eyes.setPiscarAutomatico(
        true,
        3,
        2
    );

    eyes.setModoInativo(
        true,
        2,
        4
    );
}

void loop()
{
    eyes.atualizar();
}
```

---

## WELABSDEV CristalEye

A lightweight animation engine for giving robots expressive eyes.

**C++ | Arduino | ESP32 | Adafruit GFX | OLED | Robotics**
