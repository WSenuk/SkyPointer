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

## Where to buy the parts

You can buy the main components from electronics stores such as
TinyTronics and Conrad.

- **NodeMCU ESP8266:** [Buy here](PASTE_NODEMCU_LINK_HERE)
- **28BYJ-48 + ULN2003 driver:** [Buy here](PASTE_STEPPER_LINK_HERE)
- **SG90 9g Servo:** [Buy here](PASTE_SERVO_LINK_HERE)
- **5V Power Supply:** [Buy here](PASTE_POWER_LINK_HERE)

> You do not have to buy exactly these products. Similar parts with
> the same specifications can also be used.

---

## Getting the 5V power supply

The stepper motor and servo need more power than the NodeMCU should
provide directly.

For this project I use an external **5V power supply**.

A power supply of around **5V and 2A** is enough for this prototype.

The power supply I recommend can be adjusted to different voltages.
Before connecting it to SkyPointer, make sure it is set to:

**5V**

The adapter also comes with a screw-terminal connector. This makes it
easy to connect jumper wires to the power supply.

### Connect the power supply

Connect the positive **+5V** wire to:

- ULN2003 VCC
- Servo red wire

Connect the negative **GND** wire to:

- ULN2003 GND
- Servo brown/black wire
- NodeMCU GND

The NodeMCU can stay connected to the computer using its USB cable.

> **Important:** Do not connect the external 5V supply to the NodeMCU
> 3.3V pin. Always check that the power supply is set to 5V before
> connecting the motors.
