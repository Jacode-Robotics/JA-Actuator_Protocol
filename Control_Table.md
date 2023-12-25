## Control Table
## [Instruction](#instruction)

| Value |            Instructions             |                         Description                          |
| :---: | :---------------------------------: | :----------------------------------------------------------: |
| 0x01  |          [Ping](#ins-ping)          | Instruction that checks whether the Packet has arrived to a device with the same ID as Packet ID |
| 0x02  |          [Read](#ins-read)          |           Instruction to read data from the Device           |
| 0x03  |         [Write](#ins-write)         |           Instruction to write data on the Device            |
| 0x06  |     [Factory Reset](#ins-reset)     | Instruction that resets the Control Table to its initial factory default settings |
| 0x08  |        [Reboot](#ins-reboot)        |               Instruction to reboot the Device               |
| 0x10  |         [Clear](#ins-clear)         |           Instruction to reset certain information           |
| 0x20  | [Control Table Backup](#ins-backup) | Instruction to store current Control Table status data to a Backup area or to restore EEPROM data. |
| 0x55  |    [Status(Return)](#ins-status)    |           Return packet for the Instruction Packet           |
| 0x8A  |  [Fast Sync Read](#ins-sync-read)   | For multiple devices, Instruction to read data from the same Address with the same length at once |
| 0x8B  | [Fast Sync Write](#ins-sync-write)  | For multiple devices, Instruction to write data from the same Address with the same length at once |

## [Control Table](#control-table)

| Address | Size(Byte) |                    Data Name                    | Access | Initial<br />Value |                              Range                               |          Unit           |
|:-------:|:----------:|:-----------------------------------------------:|:------:|:------------------:|:----------------------------------------------------------------:|:-----------------------:|
|    0    |     2      |      [Model Number](#Model-Numbern)      |   RW    |         -          |                                -                                 |            -            |
| 6 | 1 | [Firmware Version](#Firmware-Version) | R | - | - | - |
|    7    |     1      |                    [ID](#id)                    |   RW   |         1          | 0 ~ 252  |            -            |
|    8    |     1      |             [Baud Rate](#baud-rate)             |   RW   |         4          |                              0 ~ 4                               |            -            |
|   10    |     1      |            [Drive Mode](#drive-mode)            |   RW   |         2          |                              0 ~ 3                              |            -            |
|   11    |     1      |        [Operating Mode](#operating-mode)        |   RW   |         4          |                        4、17                        |            -            |
|   20    |     4      |         [Homing Offset](#homing-offset)         |   RW   |         0          |                0 ~<br> 32,768                |        1 [count]        |
|   31    |     1      |     [Temperature Limit](#temperature-limit)     |   RW   |         80         |                             0 ~ 100                              |       1 [&deg;C]        |
|   36    |     2      |             [PWM Limit](#pwm-limit)             |   RW   |       30,000        |                            0 ~ 37,500                             |       0.000026 [%]        |
|   44    |     4      |        [Velocity Limit](#velocity-limit)        |   RW   |       2,000        |                            0 ~ 6,000                             |     0.01 [rev/min]      |
|   48    |     4      |    [Max Position Limit](#max-position-limit)    |   RW   |      16,384       |                      -2,147,483,648 ~<br> 2,147,483,648                      |        1 [count]        |
|   52    |     4      |    [Min Position Limit](#min-position-limit)    |   RW   |      -16,384      |                      -2,147,483,648 ~<br> 2,147,483,648                      |        1 [count]        |
|   512   |     1      |          [Torque Enable](#torque-enable)          |   RW   |         0          |                        0 ~ 1                        |            -            |
|   518   |     1      |  [Hardware Error Status](#hardware-error-status)  |   R    |         0          |                          6、7                          |            -            |
|   522   |     2      |       [Movement Duration](#movement-duration)     |   RW   |        2000        |                     0 ~ 65,536                      |            1 [msec]         |
|   524   |     2      |       [Velocity I Gain](#velocity-pi-gain)        |   RW   |        0        |                     0 ~ 32,767                      |            -            |
|   526   |     2      |       [Velocity P Gain](#velocity-pi-gain)        |   RW   |        300        |                     0 ~ 32,767                      |            -            |
|   528   |     2      |       [Position D Gain](#position-pid-gain)       |   RW   |         0          |                     0 ~ 32,767                      |            -            |
|   530   |     2      |       [Position I Gain](#position-pid-gain)       |   RW   |         250          |                     0 ~ 32,767                      |            -            |
|   532   |     2      |       [Position P Gain](#position-pid-gain)       |   RW   |        9000         |                     0 ~ 32,767                      |            -            |
|   536   |     2      |   [Feedforward 2nd Gain](#feedforward-2nd-gain)   |   RW   |         20200          |                     0 ~ 32,767                      |            -            |
|   538   |     2      |   [Feedforward 1st Gain](#feedforward-1st-gain)   |   RW   |         2020          |                     0 ~ 32,767                      |            -            |
|   564   |     4      |          [Goal Position](#goal-position)          |   RW   |         -          | Min Position Limit(52) ~<br> Max Position Limit(48) |        1[pulse]         |
|   568   |     2      |          [Realtime Tick](#realtime-tick)          |   R    |         -          |                     0 ~ 32,767                      |        1 [msec]         |
|   571   |     1      |          [Moving Status](#moving-status)          |   R    |         -          |                          -                          |            -            |
|   572   |     2      |            [Present PWM](#present-pwm)            |   R    |         -          |                          -                          |       0.0096 [%]        |
|   576   |     4      |       [Present Velocity](#present-velocity)       |   R    |         -          |                          -                          |     0.0078125 [count/100us]      |
|   580   |     4      |       [Present Position](#present-position)       |   R    |         -          |                          -                          |        1 [count]        |
|   584   |     4      |    [Velocity Trajectory](#velocity-trajectory)    |   R    |         -          |                          -                          |     0.01 [rev/min]      |
|   588   |     4      |    [Position Trajectory](#position-trajectory)    |   R    |         -          |                          -                          |        1 [pulse]        |
|   594   |     1      |    [Present Temperature](#present-temperature)    |   R    |         -          |                          -                          |       1 [&deg;C]        |
|   878   |     1      |           [Backup Ready](#backup-ready)           |   R    |         -          |                        0 ~ 1                        |            -            |






# Control table description
### Firmware Version
This address stores the version number of the firmware installed on your JA actuator.
### ID
The ID identify individual actuators for instruction packets.  Values between 0 and 252 (0xFD) can be assigned to individual actuators 

> **NOTE**: JA IDs must be unique for each device connected to a JA network. Multiple devices sharing a single ID may cause communications issues or control failure. 

### Baud Rate
|   Value    |   Baud Rate   | Actual Baud Rate | Margin of Error |
|:----------:|:-------------:|:----------------:|:---------------:|
| 4(Default) |   2M [bps]    |    2,000,000     |     0.000%      |
|     3      |   1M [bps]    |    1,000,000     |     0.000%      |
|     2      | 115,200 [bps] |     115,226      |     0.023%      |
|     1      | 57,600 [bps]  |      57,613      |     0.023%      |
|     0      |  9,600 [bps]  |      9,600       |     0.000%      |

### Drive Mode

The Drive Mode control table register allows the configuration of several settings related to the movement of your JA actuator:

* Normal/Reverse mode allows the configuration of the movement direction of your JA actuator.

|     Bit     |           Item           | Description                                                                                                                                                                                                                                                                                                          |
|:-----------:|:------------------------:|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Bit 7(0x80) |            -             | Unused, always ‘0’                                                                                                                                                                                                                                                                                                   |
| Bit 6(0x40) |            -             | Unused, always ‘0’                                                                                                                                                                                                                                                                                                   |
| Bit 5(0x20) |            -             | Unused, always ‘0’                                                                                                                                                                                                                                                                                                   |
| Bit 4(0x10) |            -             | Unused, always ‘0’                                                                                                                                                                                                                                                                                                   |
| Bit 3(0x08) |            -             | Unused, always ‘0’                                                                                                                                                                                                                                                                                                   |
| Bit 2(0x04) |            -             | Unused, always ‘0’                                                                                                                                                                                                                                                                                                   |
| Bit 1(0x02) |  Disable/Enable Profile  | **[0]** Disable Profile<br />**[1]** Enable Profile                                                                                                                                                                                                                                                                                                  |
| Bit 0(0x01) |   Normal/Reverse Mode    | **[0]** Normal movement directions: Positive movement direction is counterclockwise, and negative movement direction is clockwise.<br />**[1]** Reverse Mode: Negative movement directions are counterclockwise, and positive movement directions are clockwise.                                                                                                                                                                                                              |

### Operation Mode

| Value      | Operating Mode                 | Description                                                                                                                                                                                                                   |
|:-----------|:-------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 4          | Extended Position Control Mode | This mode is similar to Position Control Mode, but is not limited by the Position Limit control table items. This allows multi turn position based control for applications requiring continuous rotation.           |
| 17         | Calibration Mode      | Self-calibrate the parameters of motor to improve the accuracy of driver recognition.  

### Homing Offset
Users can adjust the Home position by setting Home Offset(20). The Homing Offset value is added to the Present Position(580).  Present Position(580) = Actual Position + Homing Offset(20).


### Temperature Limit
This value limits operating temperature.  
When the Present Temperature(594) that indicates internal temperature of device is greater than the Temperature Limit(31), the Overheating Error Bit(0x04) in the Hardware Error Status(518) will be set.  
|Unit|Value Range|Description|
| :---: | :---: | :---: |
|About 1 [&deg;C]|0 ~ 100|0 ~ 100 [&deg;C]|

**CAUTION** : Do not set the temperature lower/higher than the default value. When the temperature alarm shutdown occurs, wait for 20 minutes to cool the temperature before reuse. Keep using the product with high temperature can cause severe damage to the device.


[Shutdown(63)]: #shutdown
[Torque Enable(512)]: #torque-enable512

### PWM Limit
This value indicates the maximum PWM output.  
Goal PWM(548) cannot be configured with any values exceeding [PWM Limit(36)].  
[PWM Limit(36)] is commonly applied in all operating mode as an output limit, therefore decreasing PWM output will also decrease torque and velocity of the device.  

[PWM Limit(36)]: #pwm-limit36

### Velocity Limit
This value indicates maximum velocity of Profile Velocity. 
[Goal Position(564)] and [Movement Duration(522)] cannot be configured with any values exceeding Velocity Limit(44). Writing invalid or the value over its limit, the Status Packet sends the Data Limit Error via its Error field.

### Max/Min Position Limit
These values limit maximum and minimum desired positions.
The Goal Position(564) and Present Position(580) can't exceed these values.
Writing invalid or the value over its limit, the Status Packet sends the Data Limit Error via its Error field. When the Present Position(580) over this limit, the Hardware Error Status(518) will be set.

### Torque Enable
Torque Enable determines Torque ON/OFF. Writing ‘1’ to Torque Enable’s address will turn on the Torque and all Data in the EEPROM area will be locked.

### Hardware Error Status
This value indicates hardware error status. The JA actuator can protect itself by detecting dangerous situations that could occur during the operation.
|  Bit  |               Item               | Description                                                                      |
|:-----:|:--------------------------------:|:---------------------------------------------------------------------------------|
| Bit 7 |           Check EEPROM Area Error           | Saving or loading system config form EEPROM area is failed                                                             |
| Bit 6 |  Position Limit Error  | Detects the Present Position  is not between Max Position Limit and Min Position Limit        |
| Bit 5 |     Overload Error               | Detects that persistent load exceeds maximum output                              |
| Bit 4 | Electrical Shock Error           | Detects electric shock on the circuit or insufficient power to operate the motor |
| Bit 3 |   Motor Encoder Error            | Detects malfunction of the motor encoder                                         |
| Bit 2 |        Overheating Error         | Detects that internal temperature exceeds the configured operating temperature   |
| Bit 1 |                -                 | Not used, always '0'                                                             |
| Bit 0 |                -                 | Not used, always '0'                                                             |

### Movement Duration
Movement Duration indicate the duration of movement in Extended Position Control Mode.

### Velocity PI Gain(524, 526), Feedforward 2nd Gains(536)
These values indicate Gains of Velocity Control Mode. Velocity P Gain of the device's internal controller is abbreviated to K<sub>V</sub>P.

|                           |  Controller Gain  |   Range    |          Description          |
|:-------------------------:|:-----------------:|:----------:|:-----------------------------:|
|   Velocity I Gain(524)    |  K<sub>V</sub>I   | 0 ~ 32,767 |    Velocity Integral Gain     |
|   Velocity P Gain(526)    |  K<sub>V</sub>P   | 0 ~ 32,767 |  Velocity Proportional Gain   |
| Feedforward 2nd Gain(536) | K<sub>FF2nd</sub> | 0 ~ 32,767 | Acceleration Feedforward Gain |

Below figure is a block diagram describing the velocity controller in Velocity Control Mode. When the instruction is received by the device, it takes following steps until driving the device.

1. An Instruction from the user is transmitted via communication bus, then registered to Goal Velocity(552).
2. Goal Velocity(552) is converted to desired velocity trajectory by Profile Acceleration(556).
3. The desired velocity trajectory is stored at Velocity Trajectory(584).
4. PI controller calculates PWM output for the motor based on the desired velocity trajectory.
5. Goal PWM(584) sets a limit on the calculated PWM output and decides the final PWM value.
6. The final PWM value is applied to the motor through an Inverter, and the device is driven.
7. Results are stored at Present Position(580), Present Velocity(576), Present PWM(572) 



>**NOTE** : K<sub>v</sub>A stands for Anti-windup Gain that cannot be modified by users. For more details about the PID controller and Feedforward controller, please refer to the [PID Controller](http://en.wikipedia.org/wiki/PID_controller) and [Feed Forward](https://en.wikipedia.org/wiki/Feed_forward_(control)).

### **Position PID Gain(528, 530, 532), Feedforward 1st Gains(538)**

These Gains are used in Position Control Mode and Extended Position Control Mode. Gains of device’s internal controller can be calculated from Gains of the Control Table as shown below. Position P Gain of device’s internal controller is abbreviated to K<sub>P</sub>P.

|                           |  Controller Gain  |   Range    |        Description         |
|:-------------------------:|:-----------------:|:----------:|:--------------------------:|
|   Position D Gain(528)    |  K<sub>P</sub>D   | 0 ~ 32,767 |  Position Derivative Gain  |
|   Position I Gain(530)    |  K<sub>P</sub>I   | 0 ~ 32,767 |   Position Integral Gain   |
|   Position P Gain(532)    |  K<sub>P</sub>P   | 0 ~ 32,767 | Position Proportional Gain |
| Feedforward 1st Gain(538) | K<sub>FF1st</sub> | 0 ~ 32,767 | Velocity Feedforward Gain  |

Below figure is a block diagram describing the position controller in Position Control Mode and Extended Position Control Mode. When the instruction is received by the device, it takes following steps until driving the device.

1. An Instruction from the user is transmitted via communication bus, then registered to Goal Position(564).
2. Goal Position(564) is converted to desired position trajectory and desired velocity trajectory by Profile Velocity(560) and Profile Acceleration(556).
3. The desired position trajectory and desired velocity trajectory is stored at Position Trajectory(588) and Velocity Trajectory(584) respectively.
4. Feedforward and PID controller calculate PWM output for the motor based on desired trajectories.
5. Goal PWM(548) sets a limit on the calculated PWM output and decides the final PWM value.
6. The final PWM value is applied to the motor through an Inverter, and the device is driven.
7. Results are stored at Present Position(580), Present Velocity(576), Present PWM(572) 

>**NOTE** : In case of PWM Control Mode, both PID controller and Feedforward controller are deactivated while Goal PWM(548) value is directly controlling the motor through an Inverter. In this manner, users can directly control the supplying voltage of the motor.

>**NOTE** : K<sub>a</sub> is an Anti-windup Gain that cannot be modified by users. For more details about the PID controller and Feedforward controller, please refer to the [PID Controller](http://en.wikipedia.org/wiki/PID_controller) and [Feed Forward](https://en.wikipedia.org/wiki/Feed_forward_(control)).

### **Goal Position**
Desired position can be set with [Goal Position(564)].  

The available input range is between [Min Position Limit(52)] and [Max Position Limit(48)] in Position Control Mode, while Extended Position Control Mode uses a value range between -2,147,483,648 ~ 2,147,483,647.

|    Angle Range     |   Value Range    |
| :----------------: | :--------------: |
| -180 [°] ~ 180 [°] | -16,384 ~ 16,384 |




### **Realtime Tick**

This value indicates device’s internal time.

|Unit       | Value Range       | Description|
| :---:    | :---:      | :---: |
| 1 [msec] | 0 ~ 32,767 | The value resets to ‘0’ when it exceeds 32,767|


### **Moving Status**

This value provides additional information about the movement. In-Position Bit(0x01) only works with Position Control Mode and Extended Position Control Mode.

|                         |      |                              Details                               |                                      Description                                       |
|:-----------------------:|:----:|:------------------------------------------------------------------:|:--------------------------------------------------------------------------------------:|
|          Bit 7          | 0x80 |                                 -                                  |                                         Unused                                         |
|          Bit 6          | 0x40 |                                 -                                  |                                         Unused                                         |
| Bit 5<br />~<br />Bit 4 | 0x30 | Profile Type(0x30)<br />Profile Type(0x10)<br />Profile Type(0x00) | Trapezoidal Velocity Profile<br />Rectangle Velocity Profile<br />Profile unused(Step) |
|          Bit 3          | 0x08 |                                 -                                  |                                         Unused                                         |
|          Bit 2          | 0x04 |                                 -                                  |                                         Unused                                         |
|          Bit 1          | 0x02 |                                 -                                  |                                         Unused                                         |
|          Bit 0          | 0x01 |                            In-Position                             |                       The device is reached to desired position                        |

### **Present PWM**
The Present PWM indicates current PWM. 
### **Present Velocity**
This value indicates the present Velocity. 
### **Present Position**
This value indicates present Position. 
### **Velocity Trajectory**
This is a desired velocity trajectory created by Profile. Operating method can be differ by control mode. For more details, please refer to the Profile Velocity(560).

**Position Control Mode, Extended Position Control Mode** : The desired Velocity Trajectory is used to create Position Trajectory(588). When Profile reaches to an endpoint, Velocity Trajectory(584) is set to ‘0’.

### **Postion Trajectory**

This is a desired position trajectory created by Profile. This value is only used in Position Control Mode and Extended Position Control Mode. 

### **Present Temperature**

This value indicates internal temperature of the device. 

### **Backup Ready**
The value in this address indicates whether the backup of the control table exists after sending the [Control Table Backup Packet](/docs/en/dxl/protocol2/#control-table-backup-0x20).

| Value | Description                    |
|:-----:|:-------------------------------|
|   0   | The backup data doesn't exist. |
|   1   | A saved backup data exists.    |
