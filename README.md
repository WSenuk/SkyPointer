# SkyPointer

## Flight tracking device for hobbyists

Simple build guide for the SkyPointer prototype.

---

## What is SkyPointer?

SkyPointer is a small physical flight tracker for people who like watching airplanes.

It uses live flight data from an online API to find the closest plane near the user. The NodeMCU reads this data and calculates where the plane is.

The product uses two motors:

- A **stepper motor** turns the pointer left or right to show the direction of the plane.
- A **servo motor** moves the pointer up or down to show how high the plane is in the sky.

When SkyPointer starts, the user first points it to **North**. After 30 seconds, the system saves that position and starts searching for nearby planes.

The goal is to make plane spotting easier because the user can simply follow the pointer and know where to look.

### System flow

![System flow](images/systemflow.png)
---

## Parts needed to create this

You need:

- NodeMCU ESP8266
- 28BYJ-48 5V stepper motor
- ULN2003 stepper driver board
- 1× 9g positional servo
- External regulated 5V power supply, around 1–2A
- Jumper wires / breadboard
- Wi-Fi or phone hotspot
- Arduino IDE

### NodeMCU ESP8266

![NodeMCU ESP8266](images/nodemcu.png)

### 28BYJ-48 5V Stepper Motor

![28BYJ-48 Stepper Motor](images/stepper-motor.png)

### 9g Positional Servo

![9g Servo Motor](images/servo.png)


