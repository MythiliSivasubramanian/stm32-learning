## 02_Reading_Electrical_Circuits.md

### User & Wake-Up Button (B1):

Let's return to the B1 User Button circuit on STM32F407VG DocID022256 Rev 6 UM1472. 

![alt text](../../screenshots/b1.png)

Below are the details as in the above diagram:
B1: USER button
-   R38: 100 Ω
-   R39: 220 kΩ
-   R35: 330 Ω
-   C38 (capacitor): 100 nF (Not Fitted)
-   PA0: STM32 input pin

Trying to understand what R38 (100 Ω) is and what it does electrically and how it relates to the button's operation.


**Capacitor (c):**
A capacitor is an electronic component that stores electrical charge temporarily. in the above image, C38 is a capacitor.
-   C is the component reference letter for a capacitor.
-   38 is its identification number in the circuit.
-   100 nF is its capacitance value. Its capacitance is measured in Farads (F).
-   Not Fitted means the PCB has a place for this capacitor, but the actual component is not installed.


**SW-PUSH CMS :**
That's the B1 push-button switch. The numbers 1, 2, 3 and 4 are the switch's terminal numbers. They identify its electrical connection points. When the button is pressed, its internal contacts connect. When it is released, they disconnect.+
This is a Normally Open (NO) push-button switch. In its released state, the switch contacts are disconnected. When pressed, the contacts connect.

**SB20 is a solder bridge, not a resistor. It allows the board designer to make or break a connection on the PCB.**