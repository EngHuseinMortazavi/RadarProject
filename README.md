# 180° Ultrasonic Radar with AVR

A low-cost embedded radar system developed using an AVR microcontroller, an HC-SR04 ultrasonic sensor, and a servo motor.

The system continuously scans an area across **180°**, measures the distance to obstacles, and visualizes their position on a radar interface running on a computer.

## Project Demo

https://github.com/your-username/your-repository/assets/your-video-id

## Overview

This project was developed as a final project for the **Microcomputer and Microcontroller Principles** course at the University of Isfahan.

The radar operates by rotating an ultrasonic sensor from **0° to 180°** and back. At each angular position, the HC-SR04 measures the distance to the nearest obstacle.

The measured angle and distance are transmitted from the AVR microcontroller to a computer through **UART at 9600 baud**. A Processing application receives the serial data and displays the detected obstacle on a graphical radar interface.

## Hardware

### Main Components

| Component          | Description                                               |
| ------------------ | --------------------------------------------------------- |
| ATmega16           | Main microcontroller used in the practical implementation |
| HC-SR04            | Ultrasonic distance sensor                                |
| Servo Motor        | Rotates the sensor through 180°                           |
| USB/UART interface | Communication with the computer                           |
| Computer           | Runs the Processing radar visualization                   |

The original design was developed around an **ATmega32**, but the practical implementation used an **ATmega16** because of component availability. The ATmega16 has less Flash and SRAM, so the OLED display and its framebuffer were removed from the practical version.

## Pin Configuration

| ATmega16 Pin | Function                      |
| ------------ | ----------------------------- |
| PD1          | HC-SR04 TRIG                  |
| PD2 / INT0   | HC-SR04 ECHO                  |
| PD5 / OC1A   | Servo PWM                     |
| UART TX      | Data transmission to computer |

The original design also used PC0/PC1 for I²C communication with the SSD1306 OLED, but these connections were removed from the practical ATmega16 implementation.

## How It Works

### 1. Ultrasonic Distance Measurement

The HC-SR04 is triggered with a short pulse. The microcontroller then measures the duration of the ECHO signal.

Timer0 is used to measure the pulse width, while external interrupt **INT0** detects the rising and falling edges of the ECHO signal.

With an 8 MHz clock and a Timer0 prescaler of 64:

```text
Timer Tick = 64 / 8 MHz
           = 8 μs
```

The measured pulse width is converted to distance approximately using:

```text
Distance (cm) ≈ Pulse Width / 58
```

The implementation also waits approximately 60 ms between measurements.

### 2. Servo Control

Timer1 generates the PWM signal required to control the servo.

The PWM frequency is:

```text
50 Hz
```

with a period of:

```text
20 ms
```

The servo position is controlled using approximately:

```text
0°   → 1000 μs
90°  → 1500 μs
180° → 2000 μs
```

The servo scans continuously from 0° to 180° and then returns from 180° to 0°.

### 3. UART Communication

For the practical implementation, the measured angle and distance are transmitted to the computer using UART.

Configuration:

```text
Baud Rate : 9600
Data      : 8 bits
Parity    : None
Stop Bits : 1
```

The transmitted data follows the format:

```text
ANGLE,DISTANCE
```

For example:

```text
90,35
```

meaning the sensor is currently at 90° and an obstacle was detected approximately 35 cm away.

### 4. Radar Visualization

A Processing application receives the UART data and converts the angle and distance into a graphical radar representation.

The Processing interface contains:

* Radar distance arcs
* Angular reference lines
* Moving scan line
* Detected obstacle indication
* Real-time serial data processing

The Processing application reads the angle and distance separated by a comma and updates the radar visualization continuously.

## Software

### AVR Firmware

The firmware was written in C and includes:

* External interrupt handling
* Timer0 overflow interrupt
* Ultrasonic distance measurement
* Timer1 Fast PWM
* Servo control
* UART communication

Main firmware functions include:

```text
ext_int0_isr()
timer0_ovf_isr()
get_distance()
servo_init()
servo_write()
uart_init()
uart_send_char()
uart_send_number()
uart_send_string()
main()
```

## Simulation

Before the practical implementation, the system was tested in **Proteus**.

The simulation included:

* ATmega16
* HC-SR04
* Servo motor
* Virtual Terminal

The simulation verified the ultrasonic measurement, servo scanning, and UART transmission before building the physical system.

## Project Evolution

The project was initially designed with:

```text
ATmega32
    +
HC-SR04
    +
Servo
    +
SSD1306 OLED
```

The practical version was modified to:

```text
ATmega16
    +
HC-SR04
    +
Servo
    +
UART
    +
Processing
```

The OLED library, I²C functions, framebuffer, and OLED drawing functions were removed to fit the project within the ATmega16's smaller memory. UART communication and Processing visualization were then added.

## Key Embedded Concepts

This project demonstrates several fundamental embedded-systems concepts:

* AVR microcontroller programming
* GPIO configuration
* External interrupts
* Timer overflow interrupts
* Timer-based pulse-width measurement
* Fast PWM
* Servo motor control
* UART communication
* Real-time sensor acquisition
* Serial data visualization
* Embedded-system simulation with Proteus

## Authors

**Hussein Mortazavi Derche**
Electrical Engineering — Control

**Mohammad Javad Tavakoli**

University of Isfahan
Department of Electrical Engineering

## References

* Microchip — ATmega16 Datasheet
* Microchip — ATmega32 Datasheet
* Microchip — AVR130: Setup and Use of AVR Timers
* AVR TWI Software Library
* SSD1306 OLED Display Library
* Processing Radar Visualization reference

## License

This project is intended for educational and academic purposes.
