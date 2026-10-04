# 2013 Toyota Tacoma Anytime Front/Rear Camera + AINAVI Head Unit Integration

## Overview

This README documents the installation and troubleshooting of an
aftermarket front/rear camera system in a **2013 Toyota Tacoma without
JBL**, using an **AINAVI Android head unit**, an **Anytime Backup
Camera-style FRONT/CENTER/REAR switch and camera switching relay**, and
the Tacoma's existing **factory rear camera displayed in the rear-view
mirror**.

The final goals were:

-   Normal radio operation in CENTER.
-   Manual FRONT camera on the AINAVI.
-   Manual REAR camera on the AINAVI.
-   Automatic aftermarket rear camera on the AINAVI when the truck is
    physically shifted into Reverse.
-   Factory mirror camera still working normally in actual Reverse.
-   No false dashboard **R** when manually selecting a camera.
-   No reverse-circuit backfeed or P0705.

The system now works as intended.

> **Important:** This documents one specific installation. Wire colors
> and aftermarket harnesses can vary. Verify circuits with a multimeter
> and use the terminal numbers printed on the relay instead of trusting
> socket wire colors.

------------------------------------------------------------------------

## Vehicle and Equipment

### Vehicle

-   2013 Toyota Tacoma
-   Non-JBL
-   Factory rear camera displayed in rear-view mirror

### Head Unit

-   AINAVI Android head unit
-   RP5-TY-101 CANBUS interface
-   AINAVI **G14 / Rear Cam Control / Backup Camera Control**
-   Rear-camera RCA video input

### Camera System

-   Aftermarket front camera
-   Aftermarket rear camera
-   Anytime Backup Camera-style FRONT/CENTER/REAR switch
-   Anytime camera video switching relay
-   Additional isolation relay

### Isolation Relay

The relay selected for the final solution was a **Dorman Conduct-Tite
88069**: - 12 V - 30 A - 5-pin - Terminals 30, 85, 86, 87, 87a - 87a
unused

------------------------------------------------------------------------

## Desired Final Behavior

  State                     AINAVI                    Factory Mirror      Dashboard
  ------------------------- ------------------------- ------------------- -----------
  CENTER / normal driving   Normal radio              Off                 Normal
  Manual REAR               Aftermarket rear camera   Off                 Normal
  Manual FRONT              Front camera              Off                 Normal
  Actual Reverse            Aftermarket rear camera   Factory camera on   R

------------------------------------------------------------------------

## Anytime Switch Wiring and Behavior

The Toyota-style switch has six wires:

  Wire     Function
  -------- -----------------------------------
  RED      +12 V power
  BLACK    Ground
  ORANGE   Camera display / override trigger
  GREEN    Front-camera selector trigger
  BLUE     Backlight +
  GRAY     Backlight -

Measured behavior:

  Position     ORANGE    GREEN
  ---------- -------- --------
  REAR         \~12 V      0 V
  CENTER          0 V      0 V
  FRONT        \~12 V   \~12 V

**ORANGE tells the AINAVI to enter camera mode.**

**GREEN tells the Anytime video relay to select the front camera.**

With GREEN at 0 V, the relay defaults to rear video.

### Built-in diode

The switch contains a diode between GREEN and ORANGE. Testing showed
approximately **0.532 V** from GREEN to ORANGE and **OL** in the reverse
direction.

``` text
GREEN --->|--- ORANGE
```

Leave this built-in diode alone. It is intentional. It also explains why
both indicators may illuminate when FRONT is selected.

------------------------------------------------------------------------

# Video Signal Path

``` text
Front Camera RCA ----\
                      >---- Anytime Video Relay ----> AINAVI Rear Camera RCA
Rear Camera RCA -----/
```

``` text
GREEN = 0 V   -> Rear video
GREEN = 12 V  -> Front video
```

The Anytime video relay and the additional isolation relay are two
different devices with two different jobs.

------------------------------------------------------------------------

# Issue 1: Manual FRONT or REAR Made the Dashboard Show R

## Symptoms

Selecting FRONT or REAR manually caused:

-   Dashboard R to illuminate even though the transmission was not in
    Reverse
-   Factory mirror camera to activate
-   Check Engine Light
-   P0705
-   The false R disappeared when the added camera wiring was
    disconnected from the Tacoma reverse circuit

## Cause

The manual ORANGE trigger was electrically tied to the Tacoma factory
Reverse circuit. When ORANGE supplied +12 V, voltage traveled backward
into the truck.

``` text
Anytime ORANGE +12 V
        |
        +------> AINAVI trigger
        |
        +------> Tacoma Reverse circuit
                    |
                    +--> Dashboard R
                    +--> Factory mirror camera
                    +--> P0705
```

## Diagnosis

Disconnecting the Tacoma factory reverse trigger from the added camera
wiring immediately stopped the false R.

## Fix

**Do not directly connect Anytime ORANGE to the Tacoma factory Reverse
+12 V circuit.**

The manual trigger and actual-Reverse trigger must be isolated.

------------------------------------------------------------------------

# Issue 2: Manual Cameras Worked, but Actual Reverse Did Not Trigger AINAVI

After removing the backfeed path:

``` text
Anytime ORANGE ---> AINAVI G14 / Rear Cam Control
```

Manual FRONT and REAR worked correctly and no longer affected the
dashboard or factory mirror.

However, shifting into actual Reverse only activated the factory mirror
camera. The AINAVI did not enter camera mode.

This showed that the AINAVI still needed a physical trigger on **G14 /
Rear Cam Control** in this installation.

------------------------------------------------------------------------

# Issue 3: Factory Mirror Reverse +12 V Triggered AINAVI, but Recreated Backfeed When Joined to ORANGE

Using the factory mirror/reverse +12 V signal to trigger AINAVI G14
successfully made the AINAVI switch to the rear camera during actual
Reverse.

But directly joining both trigger sources recreated the problem:

``` text
Factory Reverse +12 V ----+
                           +----> G14
Anytime ORANGE +12 V ------+
```

Pressing FRONT or REAR then sent ORANGE voltage backward into the
factory Reverse circuit, causing the dashboard R to return.

## Fix: Isolation Relay

A second relay was added solely to isolate the factory Reverse circuit.

### Final relay wiring

``` text
                 ISOLATION RELAY

Factory mirror /
Reverse +12 V --------------------> 86

Ground --------------------------> 85

ACC switched +12 V --------------> 30

AINAVI G14 /
Rear Cam Control <---------------- 87

87a ------------------------------ NOT USED
```

The Anytime ORANGE wire connects directly to the same AINAVI G14 node as
relay terminal 87:

``` text
                         +---- Relay 87
                         |
AINAVI G14 <-------------+
Rear Cam Control         |
                         +---- Switch ORANGE
```

### Why it works

The coil side is isolated from the contact side:

``` text
86 <---- Factory Reverse +12 V
85 <---- Ground
```

Actual Reverse energizes the relay.

The contact side then closes:

``` text
30 <---- ACC +12 V
87 ----> AINAVI G14
```

The factory Reverse signal never directly joins the manual ORANGE
trigger.

Manual ORANGE can activate G14 without feeding voltage backward into the
Tacoma Reverse circuit.

### Relay terminal summary

-   **85 -\> Ground**
-   **86 -\> Factory mirror / Reverse +12 V trigger**
-   **30 -\> ACC switched +12 V**
-   **87 -\> AINAVI G14 / Rear Cam Control**
-   **87a -\> Not used; insulate it**

Always verify the relay schematic. Some relays contain suppression
diodes, so coil polarity may matter.

------------------------------------------------------------------------

# Issue 4: Manual REAR Showed "Camera Disconnected"

After the trigger problem was solved:

-   Actual Reverse showed rear video correctly.
-   Manual REAR caused the AINAVI to enter camera mode, but it displayed
    a **camera disconnected** icon.

This meant the trigger was working but the rear camera was not providing
video.

## Test

With ignition ON, truck in Park, and manual REAR selected, voltage was
measured at the aftermarket rear camera.

Result:

**0 V**

## Cause

The rear camera was still powered by a reverse-only source.

``` text
ACTUAL REVERSE
Reverse +12 V ---> Rear camera powered ---> Video works

MANUAL REAR
ORANGE ---> AINAVI enters camera mode
Rear camera power = 0 V
AINAVI ---> "Camera disconnected"
```

## Fix

Move the aftermarket rear camera's actual power lead to **ACC switched
+12 V**.

Final camera power:

``` text
                    ACC +12 V
                       |
              +--------+--------+
              |                 |
              v                 v
       Front Camera +      Rear Camera +

Ground -> Front Camera -
Ground -> Rear Camera -
```

Now both aftermarket cameras are available whenever the
ignition/accessory circuit is on.

------------------------------------------------------------------------

# Important: RCA Trigger Wire Is Not Camera Power

The rear RCA cable has a small red wire alongside the yellow RCA.

That small wire had previously been used as a Reverse trigger. It is
**not the rear camera's actual power lead**.

Final configuration:

-   Rear camera actual RED power lead -\> ACC switched +12 V
-   Rear camera BLACK -\> Ground
-   Small red wire alongside rear RCA -\> Disconnected and insulated
-   Yellow RCA -\> Anytime video switching relay

------------------------------------------------------------------------

# Final Wiring

## ACC Power

``` text
ACC switched +12 V
        |
        +----> Anytime switch RED
        +----> Front camera +
        +----> Rear camera +
        +----> Isolation relay terminal 30
```

## Grounds

``` text
Ground
   |
   +----> Anytime switch BLACK
   +----> Front camera -
   +----> Rear camera -
   +----> Isolation relay terminal 85
```

## Manual Trigger

``` text
Anytime ORANGE ---> AINAVI G14 / Rear Cam Control
```

## Video Selection

``` text
Anytime GREEN ---> Anytime video selector relay
```

``` text
Front RCA ----\
               >---- Anytime selector ----> AINAVI camera RCA input
Rear RCA -----/
```

## Automatic Reverse Trigger

``` text
Factory mirror / Reverse +12 V ---> Isolation relay 86
ACC +12 V ------------------------> Isolation relay 30
Isolation relay 87 --------------> AINAVI G14
Isolation relay 85 --------------> Ground
Isolation relay 87a -------------> Not used
```

------------------------------------------------------------------------

# Final Combined Diagram

``` text
                       2013 TOYOTA TACOMA
                  FINAL CAMERA CONFIGURATION


                         SWITCHED ACC +12 V
                                |
             +------------------+------------------+
             |                  |                  |
             v                  v                  v
       SWITCH RED        FRONT CAMERA +      REAR CAMERA +
             |
             +---------------------------------------+
                                                     |
                                                     v
                                              RELAY TERMINAL 30


FACTORY MIRROR /
REVERSE +12 V -------------------------------> RELAY 86

GROUND --------------------------------------> RELAY 85

                                              RELAY 87
                                                 |
                                                 +---------+
                                                           |
                                                           v
SWITCH ORANGE -----------------------------------------> AINAVI G14
                                                    Rear Cam Control


SWITCH GREEN
     |
     v
ANYTIME VIDEO SELECTOR
     ^
     |
     +------------- FRONT CAMERA RCA
     |
     +------------- REAR CAMERA RCA
     |
     v
AINAVI REAR CAMERA RCA INPUT


ISOLATION RELAY 87a ------------ NOT USED

REAR RCA SMALL RED WIRE --------- NOT USED
```

------------------------------------------------------------------------

# Final Operating Logic

## CENTER

``` text
ORANGE = 0 V
GREEN  = 0 V
```

AINAVI stays in normal radio mode.

## Manual REAR

``` text
ORANGE = 12 V
GREEN  = 0 V
```

ORANGE triggers G14. AINAVI enters camera mode. GREEN remains low, so
the selector sends rear video. The rear camera is already powered from
ACC.

Result: rear camera on AINAVI with no false R and no factory mirror
activation.

## Manual FRONT

``` text
ORANGE = 12 V
GREEN  = 12 V
```

ORANGE triggers G14 and GREEN selects front video.

Result: front camera on AINAVI with no false R and no factory mirror
activation.

## Actual Reverse

Factory Reverse +12 V energizes relay 86/85. The relay closes 30 to 87,
sending ACC +12 V to G14.

With the Anytime switch in CENTER, GREEN is 0 V, so rear video is
selected.

Result:

-   AINAVI displays aftermarket rear camera
-   Factory mirror displays factory rear camera
-   Dashboard correctly shows R

------------------------------------------------------------------------

# Issue 5: AINAVI Touchscreen Went Crazy

After the camera system was working, the AINAVI touchscreen started
behaving erratically.

Symptoms:

-   Android booted normally
-   Display worked
-   Touch input produced apparent ghost touches / erratic behavior

## Fix

A **hard reboot of the AINAVI** completely resolved the touchscreen
issue.

No camera wiring changes were required.

------------------------------------------------------------------------

# Troubleshooting Summary

  -----------------------------------------------------------------------
  Problem                 Cause                   Solution
  ----------------------- ----------------------- -----------------------
  Manual FRONT/REAR       ORANGE backfeeding      Remove direct
  illuminated R           factory Reverse circuit connection

  Manual switch activated Same backfeed           Isolate factory Reverse
  factory mirror                                  trigger

  P0705                   Conflicting/backfed     Remove backfeed and
                          Reverse signal          clear code

  Actual Reverse did not  G14 lacked              Use factory Reverse
  trigger AINAVI          actual-Reverse trigger  signal through
                                                  isolation relay

  Joining Reverse +12 V   Trigger sources         Isolation relay
  and ORANGE brought      directly connected      
  false R back                                    

  Manual REAR showed      Rear camera had 0 V     Power rear camera from
  camera disconnected     outside Reverse         ACC

  Rear camera worked only Camera was              Move actual camera
  in actual Reverse       reverse-powered         power to ACC

  Touchscreen went crazy  Temporary AINAVI state  Hard reboot
                          after testing/power     
                          cycling                 
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# Quick Troubleshooting Guide

### AINAVI enters camera mode but says "camera disconnected"

Measure voltage at the selected camera.

``` text
Red probe   -> Camera +
Black probe -> Camera ground
```

Expected: approximately 12 V.

If the head unit enters camera mode but the camera has 0 V, troubleshoot
camera power before changing the trigger circuit.

### Manual camera selection makes dashboard R illuminate

Stop and inspect the trigger wiring. The manual camera trigger is
probably connected to the factory Reverse circuit.

**ORANGE should never be directly tied to factory Reverse +12 V.**

### Factory mirror activates when FRONT is selected

This is another sign of Reverse-circuit backfeed.

### Mirror works in actual Reverse but AINAVI does not

Check whether AINAVI G14 receives a trigger during actual Reverse.

### FRONT works but manual REAR does not

Check:

1.  Rear-camera power
2.  Rear-camera ground
3.  Rear RCA
4.  Anytime selector relay
5.  GREEN voltage

Remember:

``` text
GREEN = 0 V  -> Rear
GREEN = 12 V -> Front
```

------------------------------------------------------------------------

# Safety Notes

A false Reverse indication is not just a camera-display issue.

During this build, backfeeding the Reverse circuit caused an incorrect
dashboard R, factory mirror activation, and P0705.

If manually selecting a camera changes the vehicle's gear indication,
disconnect the added trigger connection and troubleshoot the isolation
before continuing.

Also:

-   Disconnect power before changing relay or head-unit wiring.
-   Fuse added power circuits appropriately.
-   Insulate unused wires and terminal 87a.
-   Keep exposed terminals/splices away from the radio chassis.
-   Verify circuits with a multimeter instead of trusting wire colors.
-   Verify relay terminals 30, 85, 86, 87, and 87a on the relay itself.

------------------------------------------------------------------------

# Key Lessons

1.  **Video selection and screen triggering are separate functions.**
    The Anytime relay chooses the video source. G14 tells the AINAVI
    when to display it.

2.  **Do not directly combine independent +12 V trigger circuits.**
    Isolation may be required.

3.  **A relay can use the factory Reverse signal without electrically
    joining it to the manual trigger circuit.**

4.  **If a camera works in Reverse but not manually, check camera
    power.** A reverse-powered camera cannot provide video during manual
    activation outside Reverse.

5.  **The small red wire alongside an RCA cable is not necessarily the
    camera's actual power wire.**

6.  **Use a multimeter.** Measurements at ORANGE, GREEN, G14, and the
    camera power leads identified the actual faults.

7.  **Change one thing at a time.** Isolating and testing individual
    circuits prevented the troubleshooting from becoming even more
    confusing.

------------------------------------------------------------------------

# Final Result

``` text
CENTER
Normal radio operation

FRONT
Manual front camera on AINAVI
No false Reverse indication
Factory mirror unaffected

REAR
Manual rear camera on AINAVI
No false Reverse indication
Factory mirror unaffected

ACTUAL REVERSE
Rear camera automatically displayed on AINAVI
Factory mirror camera operates normally
Dashboard correctly indicates Reverse
```

The successful installation required treating three functions
independently:

1.  **Camera power**
2.  **Video source selection**
3.  **Head-unit camera trigger**

Once those functions were separated and the factory Reverse circuit was
isolated with a relay, the system worked correctly.

------------------------------------------------------------------------

## Disclaimer

This README describes a specific installation on a 2013 Toyota Tacoma
with a particular aftermarket head unit and camera-switching system.
Vehicle wiring, aftermarket harnesses, relay configurations, and
head-unit pinouts may differ.

Verify your own wiring diagrams and measurements before making
connections. Incorrect connections to Reverse, transmission-range,
CANBUS, or vehicle power circuits can cause diagnostic codes or
unintended behavior.

