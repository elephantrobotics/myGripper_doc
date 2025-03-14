# Basic usage

## Serial port control method
**USB-485 module wiring**:

Connect the 24V, GND, 485_A (T/R+, 485+), 485_B (T/R-, 485-) of the smart hand end, a total of 4 wires, the power supply is a 24V DC regulated power supply, and insert the USB port of the module into the USB port of the computer

<img src="../img/new485c.png" width="100%" >

485A connects to the 485 to USB module A+;<br>
485B connects to the 485 to USB module B-;<br>
24V connects to the positive pole of the 24V DC regulated power supply;<br>
GND connects to 24V Negative pole of DC regulated power supply<br>

**Serial port debugging assistant debugging**:

Users can use [UartAssist](https://www.nitwo.com/cn/download/UartAssist.html) serial port debugging assistant, refer to the figure below to send the corresponding dexterous hand command, CRC check code does not need to be filled in, UartAssist serial port debugging assistant will automatically generate it.

<img src="../img/chuankou.png" width="70%" >

**Default configuration of the dexterous hand serial port**
- Dexterous hand ID: 14
- Baud rate: 115200
- Data bit: 8
- Stop bit: 1
- Check bit: No check bit

The dexterous hand uses a custom protocol. An instruction consists of a frame header (2byte), length (1byte), address code ID (1byte), function code (1byte), register address (2byte), register data (2*n bytes), and check code (2byte). Let's take reading the angle of the dexterous hand as an example

| Frame header | Length | ID | Function code | Register address | Register data/parameter | CRC-16MODBUS check code |
| :--- | :--- |:--- |:--- |:--- |:--- |:--- |
|Fe Fe| 08 |0e| 03 |00 0C|00 01|71 01|

**Frame header**: 254 254 <br>

**Length**: Command length 08 <br>

**ID**: 0E, can be modified in the device, the default ID is 14, 0E represents the current ID of the dexterous hand is 14 <br>

**Function code**: Identify whether the instruction is a setting or acquisition function, 06 (write operation to the register)/03 (read operation to the register). <br>

**Register address**: 00 0c Register address corresponding to the dexterous hand function <br>

**Register parameter**: 00 01 If reading, just fill in the servo ID, if setting, fill in the servo ID and the written parameters <br>

**CRC check**: 71 01 Ensure that the terminal device does not respond to data that changes during transmission, ensure the security and efficiency of the system, and the CRC check adopts the 16-bit cyclic remainder method. Verify all the hexadecimal values ​​of the frame header, length, function code, register address, and register parameters directly to get the check code 71 01.<br>

#### Command Overview
- Read the firmware major version number

  Command: fe fe 08 0e 03 00 01 00 00 72 51

  Function code: 03 Read operation

  Parameter: None

  Return: fe fe 08 0e 03 00 01 00 01 B2 90

  Note: Data return 00 01, return version number is 1

- Read the firmware minor version number

  Command: FE FE 08 0E 03 00 02 00 00 72 A1

  Function code: 03 Read operation

  Parameter: None

  Return: FE FE 08 0E 03 00 02 00 01 B2 60

  Note: Data return 00 01, indicating that the return version number is 1

- Set/read device ID number
  - Set

    Command: FE FE 08 0E 06 00 03 00 0E 76 BD

    Function code: 06 Write operation

    Parameter: 00 0E, ID setting range (1-254)

    Return: FE FE 08 0E 06 00 03 00 01 72 FD

    Note: Success returns 00 01, failure returns 00 00, after the device ID is modified, the ID in the command also needs to be modified
    to be the same to communicate

  - Read

    Command: FE FE 08 0E 03 00 04 00 00 73 41

    Function code: 03 Read operation

    Parameter: None

    Return: FE FE 08 0E 03 00 04 00 0E B7 C0

    Note: Data returns 00 0E, indicating the current smart hand ID number

- Set/read 485 baud rate

  - Set

    Instruction: FE FE 08 0E 06 00 05 00 00 73 1D

    Function code: 06 Write operation

    Parameter:  00 00, 0-115200, 1-1000000, 2-57600, 3-19200, 4-9600, 5-4800,
    If set to 1000000, the parameter is changed to 00 01

    Return: FE FE 08 0E 06 00 05 00 01 72 FD

    Note: Success returns 00 01, failure returns 00 00

  - Read

    Instruction: FE FE 08 0E 03 00 06 00 00 B3 E0

    Function code: 03 Read operation

    Parameter: None

    Return: FE FE 08 0E 03 00 06 00 00 B3 E0

    Note: Return data 00 00, corresponding to the baud rate value of the setting parameter

- Set the dexterous hand enable state

  Instruction: FE FE 08 0E 06 00 0A 00 00 B0 EC

  Function code: 06 Write operation

  Parameter: 00 00, 00 means disconnection, 00 01 means enablement

  Return: FE FE 08 0E 06 00 0A 00 01 70 2D

  Note: Success returns 00 01, failure returns 00 00

- Set/read the dexterous hand angle
  - Set

    Instruction: FE FE 0A 0E 06 00 0B 00 01 00 64 FF 39

    Function code: 06 Write operation

    Parameter: 00 01 00 64, set the servo 1 angle to 100

    Return: FE FE 08 0E 06 00 0B 00 01 B0 7C

    Note: Success returns 00 01, failure returns 00 00

  - Read

    Instruction: FE FE 08 0E 03 00 0C 00 01 71 01

    Function code: 03 Read operation

    Parameter: None

    Return: FE FE 08 0E 03 00 0C 00 64 5A C1

    Note: Return data 00 64, indicating that the current angle is 100, fully open state

- Set the servo zero position

  Command: FE FE 08 0E 06 00 0D 00 01 B1 9C

  Function code: 06 Write operation

  Parameter: 00 01 Set the servo 1 zero position

  Return: FE FE 08 0E 06 00 0D 00 01 B1 9C

  Note: Success returns 00 01, failure returns 00 00

- Read the gripping status of the dexterous hand

  Command: FE FE 08 0E 03 00 0E 00 00 71 61

  Function code: 03 Read operation

  Parameter: None

  Return: FE FE 08 0E 03 00 0E 00 01 B1 A0

  Note: 00 01, return data 0 means moving ,1: Stop motion, no clamping detected, 2: Stop motion, clamping detected, 3: Clamping detected, object dropped

- Set/read the servo P value
  - Set

    Command: FE FE 0A 0E 06 00 0F 00 01 00 64 3F C8

    Function code: 06 Write operation

    Parameter: 00 01 00 64, set the servo 1 P value to 100, setting range (1-254)

    Return: FE FE 08 0E 06 00 0F 00 01 71 3D

    Note: Success returns 00 01, failure returns 00 00
  - Read

    Command: FE FE 08 0E 03 00 10 00 01 B7 C0

    Function code: 03 Read operation
    Parameter: 00 01 Servo 1
    Return: FE FE 08 0E 03 00 10 00 64 9C 00
    Note: Return data 00 64, indicating that the current P value is 100

- Set/read the servo D value
  - Set

    Command: FE FE 0A 0E 06 00 11 00 01 00 64 3D 60

    Function code: 06 Write operation

    Parameter: 00 01 00 64, set the servo 1 D value to 100, setting range (1-254)

    Return: FE FE 08 0E 06 00 11 00 01 77 5D

    Note: Success returns 00 01, failure returns 00 00

    **If the dexterous hand shakes, the D value can be appropriately increased**
  - Read

    Command: FE FE 08 0E 03 00 12 00 01 77 61

    Function code: 03 Read operation

    Parameter: 00 01 Servo 1

    Return: FE FE 08 0E 03 00 12 00 64 5C A1

    Note: Return data 00 64, indicating that the current D value is 100

- Set/read servo I value

  - Set

    Command: FE FE 0A 0E 06 00 13 00 01 00 00 16 18

    Function code: 06 Write operation

    Parameter: 00 01 00 00, set servo 1 I value to 0, setting range (1-254)

    Return: FE FE 08 0E 06 00 13 00 01 B7 FC

    Note: Success returns 00 01, failure returns 00 00

  - Read

    Command: FE FE 08 0E 03 00 14 00 01 76 81

    Function code: 03 Read operation

    Parameter: 00 01 Servo 1

    Return: FE FE 08 0E 03 00 14 00 00 B6 40

    Note: Return data 00 00, indicating that the current I value is 0

- Set/read the clockwise running error of the servo

  - Set

    Command: FE FE 0A 0E 06 00 15 00 01 00 03 17 D0

    Function code: 06 Write operation

    Parameter: 00 01 00 03, set the error value of servo 1 to 3, setting range (0-16)

    Return: FE FE 08 0E 06 00 15 00 01 B6 1C

    Mark: Success returns 00 01, failure returns 00 00

  - Read

    Command: FE FE 08 0E 03 00 16 00 01 B6 20

    Function code: 03 Read operation

    Parameter: 00 01 Servo 1

    Return: FE FE 08 0E 03 00 16 00 01 B6 20

    Mark: Return data 00 01, indicating that the current value is 1

- Set/read the counterclockwise operational error of the servo

  - Set

    Command: FE FE 0A 0E 06 00 17 00 01 00 03 D7 A9

    Function code: 06 Write operation

    Parameter: 00 01 00 03, set the error value of the servo 3, setting range (0-16)

    Return: FE FE 08 0E 06 00 17 00 01 76 BD

    Note: Success returns 00 01, failure returns 00 00

  - Read

    Command: FE FE 08 0E 03 00 18 00 01 75 41

    Function code: 03 Read operation

    Parameter: 00 01 Servo 1

    Return: FE FE 08 0E 03 00 18 00 01 75 41

    Note: Return data 00 01, indicating that the current value is 1

- Set/read the minimum starting force of the servo

  - Set

    Command: FE FE 0A 0E 06 00 19 00 01 00 18 1D 80

    Function code: 06 Write operation

    Parameter: 00 01 00 18 Set the value of servo 1 to 24

    Return: FE FE 08 0E 06 00 19 00 01 B5 DC

    Note: Success returns 00 01, failure returns 00 00

  - Read

    Command: FE FE 08 0E 03 00 1A 00 01 B5 E0

    Function code: 03 Read operation

    Parameter: 00 01 Servo 1

    Return: FE FE 08 0E 03 00 1A 00 18 7F 21

    Note: Return data 00 18, indicating that the current value is 24

- Set/read servo torque

  - Set

    Command: FE FE 0A 0E 06 00 1B 00 01 00 64 3C F8

    Function code: 06 Write operation

    Parameter: 00 01 00 64 Set the torque of servo 1 to 100, parameter range (0-100)

    Return: FE FE 08 0E 06 00 1B 00 01 75 7D

    Note: Success returns 00 01, failure returns 00 00

  - Read

    Command: FE FE 08 0E 03 00 1C 00 01 B4 00

    Function code: 03 Read operation

    Parameter: 00 01 Servo 1

    Return: FE FE 08 0E 03 00 1C 01 2C 39 C1

- Set/read servo speed

  - Set

    Command: FE FE 0A 0E 06 00 20 00 01 00 14 1D 1C

    Function code: 06 Write operation

    Parameter: 00 01 00 14 Set the speed of servo 1 to 20, parameter range (1-100)

    Return: FE FE 08 0E 06 00 20 00 01 B8 0C

    Note: Success returns 00 01, failure returns 00 00

  - Read

    Instruction: FE FE 08 0E 03 00 21 00 01 78 91

    Function code: 03 Read operation

    Parameter: 00 01 Servo 1

    Return: FE FE 08 0E 03 00 21 00 32 6D D1

    Note: 00 32 Data returned is 50

- Set the dexterous hand gesture
  - Set

    Command: FE FE 08 0E 06 00 34 01 01 00 35 EC

    Function code: 06 Write operation

    Parameter: 01 01 Set gesture (0-4), gesture closure degree (0-5)

    Return: FE FE 08 0E 06 00 34 00 01 B8 0C

    Note: Success returns 00 01, failure returns 00 00

  - Set/read the dexterous hand angle
  - Set

    Command: FE FE 12 0E 06 00 2D 00 00 00 00 00 00 00 00 00 00 00 00 00 14 23 FC

    Function code: 06 Write operation

    Parameter: 00 00 00 00 00 00 00 00 00 00 00 00 00 14 The first 12 bytes represent the angle (0-100), and the last two bytes represent the speed (0-100)

    Return: FE FE 08 0E 06 00 2D 00 01 7B 9D

    Note: Success returns 00 01, failure returns 00 00

  - Read

    Instruction: FE FE 08 0E 03 00 32 00 00 7D A1

    Function code: 03 Read operation

    Parameter: None

    Return: FE FE 0C 0E 03 00 32 00 00 00 01 00 00 00 00 00 01 00 00 84 74

    Note: 00 00 00 01 00 00 00 00 00 01 00 00 The data returned is [0, 1, 0, 0, 1, 0]
