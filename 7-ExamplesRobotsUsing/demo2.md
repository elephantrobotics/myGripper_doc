# Mercury A1 Control

## Hardware Installation
First install the connector on the smart hand

<img src="../img/a1.png" width="100%" >

Then install another connector on the end flange of the robot arm

<img src="../img/a2.png" width="100%" >

Then install the smart hand on the end connection flange, and fix the 4 holes on the top, bottom, left and right with screws

<img src="../img/a3.png" width="100%" >

Finally, connect and tighten with M8 aviation plug wire

<img src="../img/a4.png" width="100%" >

## Software Installation
Dependency Library Installation
```bash
pip install pymycobot --upgrade
```

## Example program

```python
from pymycobot import Mercury
import time
me=Mercury()
me.set_hand_gripper_pinch_action_speed_consort(14,4,1)
me.power_on()
target_angles=[
[-11.326, 0.245, -0.909, -122.652, 5.459, 167.24, 0.38],
[-11.327, 13.361, -0.902, -122.646, 5.46, 167.24, 0.381],
[-11.319, 6.808, 16.44, -117.003, 5.461, 167.24, 0.39],
[-11.324, 18.251, 16.324, -117.001, 5.461, 167.241, 0.39]

]

def wait():
    time.sleep(0.2)
    while me.is_moving()==1:
        pass

for i in range(len(target_angles)):
    me.send_angles(target_angles[i],20)
    wait()
    if i ==1:
        me.set_hand_gripper_pinch_action_speed_consort(14,4,8)
        time.sleep(2)
        me.send_angles(target_angles[i-1],20)
        wait()
    elif i ==3:
        me.set_hand_gripper_pinch_action_speed_consort(14,4,1)
        time.sleep(2)
        me.send_angles(target_angles[i-1],20)
        wait()
```

## Effect display

<img src="../img/demo2.gif" width="90%" >