# Python USB-485 library control

## Hardware connection

Connect the 24V, GND, 485_A (T/R+, 485+), 485_B (T/R-, 485-) of the smart hand end, a total of 4 wires, the power supply is a 24V DC regulated power supply, and insert the USB port of the module into the USB port of the computer

<img src="../img/new485c.png" width="100%" >

485A connects to the 485 to USB module A+;<br>
485B connects to the 485 to USB module B-;<br>
24V connects to the positive pole of the 24V DC regulated power supply;<br>
GND connects to the negative pole of the 24V DC regulated power supply<br>

## Software installation
**Driver library installation**
[Click to download the driver library](https://github.com/elephantrobotics/Myhand)

<img src="../img/git.png" width="50%" >

##### Serial port dependency library installation
Execute the following command in the computer terminal to install the dependency library
```bash
pip install pyserial
```

## Example program

```python
from MyHand import MyGripper_H100
import time
if __name__=="__main__":
    hand=MyGripper_H100("COM8")
    hand.set_gripper_pose(0,0)
    time.sleep(2)
    hand.set_gripper_pose(1,5)
    time.sleep(5)
    hand.set_gripper_pose(2,5)
    time.sleep(5)
    hand.set_gripper_pose(3,5)
    time.sleep(5)
    hand.set_gripper_pose(4,15)
    time.sleep(5)
    hand.set_gripper_pose(0,0)
    time.sleep(2)
```
## Effect display

<img src="../img/demo1.gif" width="90%" >

