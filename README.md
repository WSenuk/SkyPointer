# SkyPointer
**Flight tracking device for hobbyists**

Simple build guide for the SkyPointer prototype.

---

## What is SkyPointer?
SkyPointer is a small physical flight tracker for people who like watching airplanes.

It uses live flight data from an online API to find the closest plane near the user. The NodeMCU reads this data and calculates where the plane is.

The product uses two motors:
- A **stepper motor** turns the pointer left or right to show the direction of the plane.
- A **servo motor** moves the pointer up or down to show how high the plane is in the sky.

When SkyPointer starts, the user first points it to North. After 30 seconds, the system saves that position and starts searching for nearby planes.

The goal is to make plane spotting easier because the user can simply follow the pointer and know where to look.

### System flow
![System flow](images/systemflow.png)

---

## Parts needed to create this
To build SkyPointer you need the following parts:
- NodeMCU ESP8266
- 28BYJ-48 5V stepper motor
- ULN2003 stepper driver board
- 1× 9g positional servo
- External regulated 5V power supply, around 1–2A
- Jumper wires / breadboard
- Wi-Fi or phone hotspot
- Arduino IDE

### Main components
<table>
  <tr>
    <td align="center">
      <img src="images/nodemcu.png" width="220"><br>
      <b>NodeMCU ESP8266</b>
    </td>
    <td align="center">
      <img src="images/stepper-motor.png" width="220"><br>
      <b>28BYJ-48 Stepper Motor</b>
    </td>
    <td align="center">
      <img src="images/servo.png" width="220"><br>
      <b>9g Servo Motor</b>
    </td>
  </tr>
</table>

The ULN2003 driver board is used between the NodeMCU and the stepper motor. The 28BYJ-48 stepper motor normally plugs directly into this driver board.

### Where to buy the parts
You can buy these parts from electronics shops such as TinyTronics or Conrad.

- **NodeMCU ESP8266:** [TinyTronics](https://www.tinytronics.nl/en/development-boards/microcontroller-boards/with-wi-fi/esp8266-nodemcu-v2)
- **28BYJ-48 Stepper Motor + ULN2003 driver:** [Conrad](https://www.conrad.nl/nl/p/whadda-wpi401-stappenmotorbesturingsmodule-geschikt-voor-serie-arduino-1-stuk-s-2330784.html)
- **SG90 9g Servo Motor:** [TinyTronics](https://www.tinytronics.nl/en/mechanics-and-actuators/motors/servomotors/sg90-mini-servo)
- **External power supply:** [TinyTronics adjustable power supply](https://www.tinytronics.nl/nl/power/voedingen/12v/goobay-64570-universele-voedingsadapter-verstelbaar-3-12v-2.25a)

You do not have to buy exactly these products. Similar components with the same specifications can also be used.

### Getting the 5V power supply
The stepper motor and servo need their own power supply.

The NodeMCU should not power both motors directly because the motors can use more current than the NodeMCU can safely provide.

For this prototype you can use an external power supply of around **5V and 1–2A**.

An adjustable power supply can also be used. Before connecting it to the prototype, make sure it is set to **5V**.

The positive **5V** connection will later be connected to the stepper motor driver and the servo motor.

The **GND** connection will also be connected to the NodeMCU so that all parts share the same ground.

The exact wiring is explained later in the wiring section.

---

## Software setup

### Install the ESP8266 board package
Before we start, we need to install the Arduino IDE and connect the NodeMCU to the computer.

I have added a PDF guide that explains how to install the Arduino IDE and how to connect the NodeMCU to your device.

### Install ArduinoJson
Now we can start with the Arduino program.

Open the **Arduino IDE**.

On the left side you will see an icon that looks like a group of books. Click on this icon to open the **Library Manager**.

![Open the Library Manager](images/arduinojson-1.png)

In the search bar, search for **ArduinoJson by Benoit Blanchon**.
ArduinoJson is used to read the aircraft data that comes from the API.

![Search for ArduinoJson](images/arduinojson-2.png)

Click **Install**.

![Install ArduinoJson](images/arduinojson-3.png)

ArduinoJson is now installed.

### Install AccelStepper
Search for AccelStepper — by Mike McCauley Used to control the 28BYJ-48 stepper motor.
Click on install. 
![Install AccelStepper](images/accelstepper.png)

### Adjusting the code
Now that the libraries are installed, we need to adjust the code so it uses your Wi-Fi and your current location.
#### The Code
Click on **File** in the top-left corner of the Arduino IDE.
Then click **New Sketch**.
This will open a new place where you can add your code.
Copy the code below and paste it into the new sketch.
<details>
<summary><b>Click here to show the SkyPointer code</b></summary>
  
```cpp
#include <ESP8266WiFi.h>
#include <WiFiClientSecure.h>
#include <ESP8266HTTPClient.h>
#include <ArduinoJson.h>
#include <AccelStepper.h>
#include <Servo.h>

// =====================================================
// WIFI
// =====================================================

#define WIFI_SSID "YOUR_WIFI_NAME"
#define WIFI_PASSWORD "YOUR_WIFI_PASSWORD"


// =====================================================
// YOUR LOCATION
// =====================================================

const float MY_LAT = 52.049380;
const float MY_LON = 5.278542;

// Search radius in nautical miles
const int RADIUS = 50;


// =====================================================
// STEPPER MOTOR
// 28BYJ-48 + ULN2003
// =====================================================

#define IN1 D1
#define IN2 D2
#define IN3 D5
#define IN4 D6

// Around one full rotation
const long STEPS_PER_REVOLUTION = 4096;

AccelStepper directionStepper(
  AccelStepper::HALF4WIRE,
  IN1,
  IN3,
  IN2,
  IN4
);


// =====================================================
// SERVO MOTOR
// =====================================================

Servo skyServo;


// =====================================================
// INTERNET
// =====================================================

WiFiClientSecure client;


// =====================================================
// SETUP
// =====================================================

void setup()
{
  Serial.begin(115200);
  Serial.println();

  // ---------------------------------
  // STEPPER SETUP
  // ---------------------------------

  directionStepper.setMaxSpeed(500);
  directionStepper.setAcceleration(300);

  // Release the stepper motor
  // so the pointer can be set to North
  directionStepper.disableOutputs();


  // ---------------------------------
  // SERVO SETUP
  // ---------------------------------

  skyServo.attach(D7);

  // Start at the horizon
  skyServo.write(0);


  // ---------------------------------
  // CONNECT TO WIFI
  // ---------------------------------

  Serial.print("Connecting to WiFi");

  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);

  while (WiFi.status() != WL_CONNECTED)
  {
    delay(500);
    Serial.print(".");
  }

  Serial.println();
  Serial.println("WiFi connected!");

  client.setInsecure();


  // =================================================
  // NORTH SETUP
  // =================================================

  Serial.println();
  Serial.println("=========================");
  Serial.println("POINT THE ARROW NORTH");
  Serial.println("=========================");
  Serial.println();
  Serial.println("Turn the pointer so it");
  Serial.println("points North.");
  Serial.println();
  Serial.println("You have 30 seconds.");
  Serial.println();


  // 30 second countdown
  for (int i = 30; i > 0; i--)
  {
    Serial.print("Starting in ");
    Serial.print(i);
    Serial.println(" seconds");

    delay(1000);
  }


  // Current position becomes North
  directionStepper.setCurrentPosition(0);

  // Turn stepper motor back on
  directionStepper.enableOutputs();


  Serial.println();
  Serial.println("=========================");
  Serial.println("NORTH SAVED!");
  Serial.println("0 degrees = North");
  Serial.println("=========================");
  Serial.println();

  delay(1000);

  Serial.println("Searching for closest plane...");
}


// =====================================================
// LOOP
// =====================================================

void loop()
{
  if (WiFi.status() == WL_CONNECTED)
  {
    getClosestPlane();
  }

  Serial.println();
  Serial.println("Next update in 30 seconds...");
  Serial.println();

  delay(30000);
}


// =====================================================
// GET CLOSEST PLANE
// =====================================================

void getClosestPlane()
{
  HTTPClient https;

  String url =
    "https://api.adsb.lol/v2/closest/" +
    String(MY_LAT, 6) + "/" +
    String(MY_LON, 6) + "/" +
    String(RADIUS);

  Serial.println();
  Serial.println("Searching for closest plane...");

  if (https.begin(client, url))
  {
    int httpCode = https.GET();


    // ---------------------------------
    // API WORKED
    // ---------------------------------

    if (httpCode == 200)
    {
      String payload = https.getString();

      DynamicJsonDocument doc(8192);

      DeserializationError error =
        deserializeJson(doc, payload);


      // JSON ERROR
      if (error)
      {
        Serial.println("JSON error!");

        https.end();
        return;
      }


      JsonArray aircraft = doc["ac"];


      // ---------------------------------
      // PLANE FOUND
      // ---------------------------------

      if (aircraft.size() > 0)
      {
        JsonObject plane = aircraft[0];


        // ---------------------------------
        // PLANE DATA
        // ---------------------------------

        const char* callsign =
          plane["flight"] | "Unknown";

        const char* registration =
          plane["r"] | "Unknown";

        const char* type =
          plane["t"] | "Unknown";

        float planeLat =
          plane["lat"] | 0.0;

        float planeLon =
          plane["lon"] | 0.0;

        float speed =
          plane["gs"] | 0.0;


        // =================================================
        // ALTITUDE
        // =================================================

        float altitudeFeet = 0;

        if (plane["alt_baro"].is<const char*>())
        {
          String altitudeText =
            plane["alt_baro"].as<const char*>();

          if (altitudeText == "ground")
          {
            altitudeFeet = 0;
          }
        }
        else
        {
          altitudeFeet =
            plane["alt_baro"] | 0.0;
        }


        // =================================================
        // DISTANCE
        // =================================================

        float distanceKm =
          calculateDistance(
            MY_LAT,
            MY_LON,
            planeLat,
            planeLon
          );


        // =================================================
        // DIRECTION / BEARING
        // =================================================

        float bearing =
          calculateBearing(
            MY_LAT,
            MY_LON,
            planeLat,
            planeLon
          );


        // =================================================
        // SKY ELEVATION
        // =================================================

        float altitudeMeters =
          altitudeFeet * 0.3048;

        float distanceMeters =
          distanceKm * 1000;

        float elevationAngle =
          atan2(
            altitudeMeters,
            distanceMeters
          ) * 180.0 / PI;


        elevationAngle =
          constrain(
            elevationAngle,
            0,
            90
          );


        // =================================================
        // MOVE DIRECTION STEPPER
        // =================================================

        moveStepperToBearing(bearing);


        // =================================================
        // MOVE SKY SERVO
        // =================================================

        int skyAngle =
          (int)elevationAngle;

        skyServo.write(skyAngle);


        // =================================================
        // SERIAL MONITOR
        // =================================================

        Serial.println();
        Serial.println("-------------------------");
        Serial.println("CLOSEST AIRCRAFT");
        Serial.println("-------------------------");

        Serial.print("Callsign: ");
        Serial.println(callsign);

        Serial.print("Registration: ");
        Serial.println(registration);

        Serial.print("Aircraft type: ");
        Serial.println(type);

        Serial.print("Altitude: ");
        Serial.print(altitudeFeet);
        Serial.println(" ft");

        Serial.print("Speed: ");
        Serial.print(speed);
        Serial.println(" knots");

        Serial.println();

        Serial.print("Distance: ");
        Serial.print(distanceKm);
        Serial.println(" km");

        Serial.print("Bearing from North: ");
        Serial.print(bearing);
        Serial.println(" degrees");

        Serial.print("Sky elevation: ");
        Serial.print(elevationAngle);
        Serial.println(" degrees");

        Serial.println();

        Serial.print("Stepper points to: ");
        Serial.print(bearing);
        Serial.println(" degrees");

        Serial.print("Sky servo: ");
        Serial.print(skyAngle);
        Serial.println(" degrees");

        Serial.println("-------------------------");
      }


      // ---------------------------------
      // NO PLANE FOUND
      // ---------------------------------

      else
      {
        Serial.println("No aircraft found nearby.");
      }
    }


    // ---------------------------------
    // API ERROR
    // ---------------------------------

    else
    {
      Serial.print("HTTP error: ");
      Serial.println(httpCode);
    }


    https.end();
  }


  // ---------------------------------
  // CONNECTION ERROR
  // ---------------------------------

  else
  {
    Serial.println("Could not connect to API.");
  }
}


// =====================================================
// MOVE STEPPER TO PLANE DIRECTION
// =====================================================

void moveStepperToBearing(float bearing)
{
  // Convert 0 - 360 degrees
  // into motor steps

  long targetSteps =
    (bearing / 360.0) *
    STEPS_PER_REVOLUTION;


  // Find current position
  // inside one full rotation

  long currentSteps =
    directionStepper.currentPosition()
    % STEPS_PER_REVOLUTION;


  if (currentSteps < 0)
  {
    currentSteps +=
      STEPS_PER_REVOLUTION;
  }


  // Calculate how far to move

  long difference =
    targetSteps - currentSteps;


  // Take the shortest way around

  if (difference >
      STEPS_PER_REVOLUTION / 2)
  {
    difference -=
      STEPS_PER_REVOLUTION;
  }


  if (difference <
      -STEPS_PER_REVOLUTION / 2)
  {
    difference +=
      STEPS_PER_REVOLUTION;
  }


  // Move the motor

  directionStepper.move(difference);

  directionStepper.runToPosition();
}


// =====================================================
// CALCULATE DISTANCE
// =====================================================

float calculateDistance(
  float lat1,
  float lon1,
  float lat2,
  float lon2
)
{
  const float earthRadius = 6371.0;

  float dLat =
    radians(lat2 - lat1);

  float dLon =
    radians(lon2 - lon1);

  lat1 = radians(lat1);
  lat2 = radians(lat2);


  float a =
    sin(dLat / 2) *
    sin(dLat / 2) +

    cos(lat1) *
    cos(lat2) *
    sin(dLon / 2) *
    sin(dLon / 2);


  float c =
    2 * atan2(
      sqrt(a),
      sqrt(1 - a)
    );


  return earthRadius * c;
}


// =====================================================
// CALCULATE DIRECTION TO PLANE
// =====================================================

float calculateBearing(
  float lat1,
  float lon1,
  float lat2,
  float lon2
)
{
  lat1 = radians(lat1);
  lat2 = radians(lat2);

  float dLon =
    radians(lon2 - lon1);


  float y =
    sin(dLon) *
    cos(lat2);


  float x =
    cos(lat1) *
    sin(lat2) -

    sin(lat1) *
    cos(lat2) *
    cos(dLon);


  float bearing =
    degrees(
      atan2(y, x)
    );


  if (bearing < 0)
  {
    bearing += 360;
  }


  return bearing;
}
```
#### Adding Wifi
Once the code is copied in. We need to connect the NodeMCU with your wifi. The NodeMCU works with a 2.4ghz connection. Turn on the hotspot on your phone so that the NodeMCU can connect to it. 
We need to give it access to your wifi credentials. Add your wifi name and password between the brackets( "YOUR_WIFI_NAME", "YOUR_WIFI_PASSWORD" ).
```cpp
#define WIFI_SSID "YOUR_WIFI_NAME"
#define WIFI_PASSWORD "YOUR_WIFI_PASSWORD"
```

## Wiring

### Connecting the stepper motor

### Connecting the servo motor

### Connecting the 5V power supply

---

## Testing the prototype

### Testing the flight data

### Testing the stepper motor

### Testing the servo motor

---

## Problems and solutions

---

## Sources
