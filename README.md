#robotanno-commands
# RobotAnno Controller Command Set Documentation

## 1. Serial Port Command Description

### 1.1 Serial Port Mode
Serial Port Parameters: Baud rate 115200; Data bits 8; Stop bits 1.

Command Format:
Control commands are in hexadecimal: 
- `0X10` Idle Mode
- `0X11` File Mode
- `0X12` Homing Mode
- `0X13` Run Mode
- `0X14` Debug Mode
- `0X15` Reset Mode
- `0X05` Query Mode

Run commands all start with `GXX` and end with carriage return and newline.
Teach commands end with carriage return and newline.

For system versions 180521 and above, when the parameter file sets _flagSoftRst=1, serial port 1 can send `0X18` under any circumstances to make output port o14 output. This can be used to control NRST reboot (emergency stop) by connecting a relay to o14.

#### `0X05`: Query Mode
In any working mode of the control card (except file mode), sending 0X05 via the serial port can query the current working mode of the control card. The return values can be 0X10, 0X12, 0X13, 0X14, or 0X15.

#### `0X10`: Enter Idle Mode
When the control card is in idle mode (`0X10`), it remains in idle mode.

- In file mode, it takes effect after the file operation command is completed and `0X10` is sent.
- In homing mode, it takes effect after the robot stops moving (send `0X30`) and `0X10` is sent.
- In run mode, it takes effect after the robot stops moving (send `0X30`) and 0X10 is sent.
- In reset mode, it exits to idle mode after the reset is completed. It starts up in idle mode.

#### `0X11`: Enter File Mode
To download and execute a file, select the file and click `Send File` after sending `0X11`. The first line of the file:

| Command    | Description                              |
|------------|------------------------------------------|
| `wr=ST`    | Run file                                  |
| `WR=INI`   | Parameter file                            |
| `WR=oq`    | Zero return file                          |
| `WR=T_0`   | Default run file (equivalent to `wr=ST`)  |
| `WR=T_1`   | Run 1 file                                |
| `WR=T_2`   | Run 2 file                                |
| ...        | ...                                       |
| `WR=T_120` | Run 120 file                              |


The second line is the file name; 

the third line is the file length, 
the size of the file excluding the first three lines.

To read the run file after sending `0X11`, send `WR=R,##` (`##` is the length of the read content) to receive the run file.

To read the parameter file after sending `0X11`, send `WR=P,##` (`##` is the length of the read content) to receive the parameter file.

To read the zero return file after sending `0X11`, send `WR=Q,##` (`##` is the length of the read content) to receive the zero return file.

To read the run 0 file after sending `0X11`, send `WR=H_0,##` (`##` is the length of the read content) to receive the run 0 file.

To read the run 1 file after sending `0X11`, send `WR=H_1,##` (`##` is the length of the read content) to receive the run 1 file.
...
To read the run 120 file after sending `0X11`, send `WR=H_120,##` (`##` is the length of the read content) to receive the run 120 file.

#### `0X12`: Motor Homing Mode
After entering homing mode, the system automatically runs the contents of the file and commands sent via the serial port in real time. 
| Command   | Description          |
|-----------|----------------------|
| `0X30`    | Pause command.       |
| `0X13`    | Continue command.    |
| `0X10`    | Exit command.        |

Instructions can be obtained from the run file or received via the serial port.
After running the file content, it automatically exits to idle mode.

#### `0X13`: Enter Run Mode
After entering run mode, the system automatically runs the contents of the file and commands sent via the serial port in real time. 
| Command   | Description          |
|-----------|----------------------|
| `0X30`    | Pause command.       |
| `0X13`    | Continue command.    |
| `0X10`    | Exit command.        |

Instructions can be obtained from the run file or received via the serial port.
After running the file content, it automatically starts running from the beginning.

#### `0X14`: Enter Debug Mode
| Command               | Description                                               |
|-----------------------|-----------------------------------------------------------|
| `0X05`                | Query mode.                                               |
| `0X10`                | Exit mode.                                                |
| `0`                   | Stop and send machine axis angles.                         |
| `SVON=1`              | Enable servo.                                             |
| `SVON=0`              | Disable servo.                                            |
| `SVBM=1`              | Release servo brake.                                      |
| `SVBM=0`              | Engage servo brake.                                       |
| `CLR=ST_PUL`          | Clear position on the control card.                        |
| `CLR=SV_ALM`          | Clear drive alarms.                                       |
| `J#+`                 | Move axis # in positive direction (# = 1, 2, 3, 4, 5, 6). |
| `J#-`                 | Move axis # in negative direction.                         |
| `J#0`                 | Stop axis # motion.                                        |
| `J#<`                 | Repeat motion for axis # (@ 0° to -90°).                   |
| `J#>`                 | Repeat motion for axis # (@ 0° to 90°).                    |
| `P VE=####`           | Change running speed to #### pulses per second.            |
| `P AC=####`           | Change acceleration to #### pulses per square second.      |
| `P DE=####`           | Change deceleration to #### pulses per square second.      |
| `G00 J#=####,....`    | Move robot joints to specified angles.                     |
| `G07 **=####`         | Change specified robot parameters.                         |
| `TX+`, `TX-`, `TY+`, `TY-`, `TZ+`, `TZ-`, `TA+`, `TA-`, `TB+`, `TB-`, `TC+`, `TC-` | Cartesian coordinate movements. |
| `P CYCP1=800000`      | J1 axis subdivided into 800000 pulses per revolution.      |
| `P J1=24000`          | Assigning value to J1.                                    |
| `P=VE`                | Query VE value. Available queries include VE, AC, DE, CYCP1, ..., J1, ..., H0, D1, H2, ... |


#### `0X15`: Enter Reset Mode
Return the machine to the zero position.

## 2. Button Control Instructions

- On startup, pressing the `Start` button puts the robot into run mode.
- In run mode:
  - While the robot is running, pressing the `Stop` button pauses the robot.
  - When the robot is stopped, pressing the `Start` button resumes its operation.
  - When the robot is stopped, holding down the `Stop` button and then pressing the `Start` button will reset the robot and exit to idle mode.

## Instruction details:

G0n=GGN, such as G06 O=P1.1 is equivalent to GG6 O=P1.1

#### G00, G20 Point-to-point commands 
| Command                                       | Description                                             |
|-----------------------------------------------|---------------------------------------------------------|
| `G00 J1=0 J2=0 J3=-90 J4=0 J5=-90 J6=0`        | Absolute joint position with acceleration/deceleration  |
| `G00 J1=VXX J2=0 J3=VXX J4=VXX J5=-90 J6=0`    | Absolute joint position with acceleration/deceleration  |
| `G00 ... J2'4 J3'VXX ...`                     | Relative joint position command with acceleration/deceleration |
| `G00 ... J2'4 J3'VXX J4=VXX J5=-90...`        | Description of J2                                       |
| `G20 X=300 Y=100 Z=500 A=0 B=180 C=0 D=0`     | With acceleration and deceleration                      |
| `G20 X=300 Y=VXX Z=VXX A=VXX B=180 C=VXX D=0` | With acceleration and deceleration                      |
| `G30 EQ01 J1=UXXXX,J2=UXXXX,J3=UXXXX`         | Read from memory                                        |
| `G30 EQ01 J1=U30100 J2=U30120 J3=U30140 J4=U30160` | Read from memory                                |
| `G30 EQ02 U3060=XX.XX`                        | Write to memory                                         |
| `G30 EQ02 U30500=10`                          | Write to memory                                         |
| `G30 EQ03 ASK=UXXXX`                          | Query memory content                                    |
| `G30 EQ04 Vxx=UXXXXXX`                        | Description of Vxx                                      |
| `G30 EQ05 UXXXXXX=Vxx`                        | Description of UXXXXXX                                  |
| `G40 X=300 Y=100 Z=500 A=0 B=180 C=0 D=0`     | Without acceleration and deceleration                   |


#### G01 G21 Linear command 
| Command                                       | Description                                             |
|-----------------------------------------------|---------------------------------------------------------|
| `G01 J1=0 J2=0 J3=-90 J4=0 J5=-90 J6=0`        | Linear command with joint positions                     |
| `G21 X=300 Y=100 Z=500 A=0 B=180 C=0 D=0`       | Linear command in millimeters with acceleration/deceleration |
| `G21 X=VXX Y=VXX Z=500 A=0 B=VXX C=0 D=0`       | Linear command with variables and acceleration/deceleration |
| `G41 X=300 Y=100 Z=500 A=0 B=180 C=0 D=0`       | Linear command with acceleration and deceleration        |
| `G41 X=300 Y=VXX Z=VXX A=0 B=180 C=0 D=0`       | Linear command with variables and acceleration/deceleration |


### G02 G03 G04 G22 G23 G06 DEGREE=ARC (or degree) Arc command 
When DEGREE=ARC, it walks along a three-point arc

Current point: G21 X=200 Y=0 Z=200 A=-180 B=150 C=0 D=0

Second point: G22 X=300 Y=100 Z=200 A=-180 B=150 C=90 D=0

Third point: G23 X=400 Y=0 Z=200 A=-180 B=150 C=300 D=0

Example: `G06 DEGREE=300`


#### G06 Command:
| Command                     | Description                                                                 |
|-----------------------------|-----------------------------------------------------------------------------|
| `G06 T=500`                 | Delay 500 milliseconds                                                      |
| `G06 t=VXX`                 | Delay VXX milliseconds (e.g., `G08 MOV V400=#2000 G06 t=V400` for 2000 ms delay) |
| `G06 I=P1.1`                | Wait for input port P1 to be high                                           |
| `G06 I=P2.0`                | Wait for input port P2 to be low                                            |
| `G06 O=P1.1`                | Set output port P1 to high                                                  |
| `G06 O=P2.0`                | Set output port P2 to low                                                   |
| `G06 SCAN=I`                | Read input port values, store in V144-V150 cells                            |
| `G06 SCAN=O`                | Read output port values, store in V160-V166 cells                           |
| `G06 SCAN=RTC`              | Read system clock values, store in V176-V179 cells                          |
| `G06 DEGREE=ARC`            | Robot walks a three-point arc                                               |
| `G06 DEGREE=35.2`           | Robot walks 35.2 degrees                                                    |
| `G06 REPOS=J#`              | J# axis finds HOME# sensor in increasing angle direction (returns to low level) |
| `G06 REPOS=-J#`             | J# axis finds HOME# sensor in decreasing angle direction (returns to low level) |
| `G06 REPOS=J1`              | J1 axis finds HOME0 sensor in increasing angle direction (returns to low level) |
| `G06 REPOS=-J1`             | J1 axis finds HOME0 sensor in decreasing angle direction (returns to low level) |
| `G06 REPOS=JH#`             | J# axis finds HOME# sensor in increasing angle direction (returns to high level) |
| `G06 REPOS=-JH#`            | J# axis finds HOME# sensor in decreasing angle direction (returns to high level) |
| `G06 REPOS=JH1`             | J1 axis finds HOME0 sensor in increasing angle direction (returns to high level) |
| `G06 REPOS=-JH1`            | J1 axis finds HOME0 sensor in decreasing angle direction (returns to high level) |


#### G07 Command
| Command               | Description                                            |
|-----------------------|--------------------------------------------------------|
| `G07 VE=250000`       | Velocity: 250,000 pulses per second                    |
| `G07 AC=250000`       | Acceleration: 250,000 pulses per second squared        |
| `G07 DE=250000`       | Deceleration: 250,000 pulses per second squared        |
| `G07 VPP=250000`      | Maximum Velocity: 250,000 pulses per second            |
| `G07 VP=20`           | Speed: VE is 20% of VPP, VE = VPP * VP * 0.01           |
| `G07 vp=VXX`          | Speed: VE is the value in unit VXX, VE = VPP * value in VXX unit * 0.01 |
| `G07 _h0=xx`          | Height Change                                          |
| `G07 RCM=1`           | Print running commands: 1 for yes, 0 for no             |
| `G07 GCM=1`           | Teach in Cartesian coordinates: 1 for yes, 0 for joint coordinates |
| `G07 MARKPOS_HERE`    | Template MARK point coordinates: X1=0 Y1=0 X2=100 Y2=100 |
| `G07 MARKPOS`         | Matching MARK point coordinates: X1=10 Y1=10 X2=110 Y2=110 |
| `G07 UNIT=1.0`        | Interpolation precision (units in mm)                  |
| `G07 P J#=XXXX`       | Calibrate axis J# angle                                |


#### G08 Command
| Command                    | Description                                                                                   |
|----------------------------|-----------------------------------------------------------------------------------------------|
| `G08 XXXX`                 | Label (Labels cannot be followed by comments, label length must be less than 15 bytes)         |
| `G08 ACALL XXXX`           | Call XXXX label until `G08 END`                                                               |
| `G08 END`                  | End of call                                                                                   |
| `G08 AJMP XXXX`            | Jump to XXXX and execute                                                                      |
| `G08 FLTAB=#`              | Jump to file T_# and execute                                                                  |
| `G08 IF VXX>=VXX ACALL XXXX`| Integer comparison immediately for # comparison >=, <=, ==, !=, >, < (`G08 IF VXX>=VXX AJMP XXXX`) |
| `G08 IF_ELSE VXX>=VXX? ACALL XXXX: ACALL XXXX` | Conditional integer comparison, true value runs before colon, false value runs after colon (`ACALL` is call, `AJMP` is jump) |
| `G08 IFF`                  | Floating point comparison                                                                     |
| `G08 IF_ELSEF`             | Floating point conditional comparison                                                         |
| `G08 MOV VXX=#XX`          | Integer transfer (immediate number `G08 MOV V10=#12`) (unit `G08 MOV V10=V12`)                 |
| `G08 MOVF VXX=#XX`         | Floating point transfer (immediate number `G08 MOVF V10=#12.5`) (unit `G08 MOVF V10=V12`)      |
| `G08 PRINT VXX`            | Print the value of VXX in integer form                                                        |
| `G08 PRINTF VXX`           | Print the value of VXX in floating point form                                                 |
| `G08 INT VXX`              | Convert floating point to integer                                                             |
| `G08 FLOAT VXX`            | Convert integer to floating point                                                             |
| `G08 ADD VXX=VXX+VXX`      | Integer addition (immediate number #)                                                         |
| `G08 SUBB VXX=VXX-VXX`     | Integer subtraction (immediate number #)                                                      |
| `G08 MUL VXX=VXX*VXX`      | Integer multiplication (immediate number #)                                                   |
| `G08 DIV VXX=VXX/VXX`      | Integer division (immediate number #)                                                         |
| `G08 SQRT VXX=VXX`         | Integer square root (immediate number #)                                                      |
| `G08 ADDF VXX=VXX+VXX`     | Floating point addition (immediate number #)                                                  |
| `G08 SUBBF VXX=VXX-VXX`    | Floating point subtraction (immediate number #)                                               |
| `G08 MULF VXX=VXX*VXX`     | Floating point multiplication (immediate number #)                                            |
| `G08 DIVF VXX=VXX/VXX`     | Floating point division (immediate number #)                                                  |
| `G08 SQRTF VXX=VXX`        | Floating point square root (immediate number #)                                               |
| `G08 STO`                  | Program automatically pauses                                                                 |
| `G08 EXIT`                 | Exit running and enter idle mode                                                              |



#### G09 Command:
| Command             | Description                                                                                      |
|---------------------|--------------------------------------------------------------------------------------------------|
| `G09 COPYRIGHT`     | Ask system YN                                                                                    |
| `G09 COM2=XXXX`     | COM2 port sends XXXX characters                                                                  |
| `Example:`          | `G09 COM2=G00 J1=10 J2=10 J3=-90 J4=0 J5=0 J6=0`                                                  |
|                     | This controller sends commands to another controller COM1,                                        |
| `G09 COM2 10H`      | Serial port 2 sends 0X10                                                                         |
| `G09 COM2 12H`      | Serial port 2 sends 0X12                                                                         |
| `G09 COM2 13H`      | Serial port 2 sends 0X13                                                                         |
| `G09 COM2 14H`      | Serial port 2 sends 0X14                                                                         |
| `G09 COM2 15H`      | Serial port 2 sends 0X15                                                                         |
| `G09 COM2 18H`      | Serial port 2 sends 0X18                                                                         |
| `G09 ENC....`       | Special instruction for connecting with absolute value servo motors, refer to the homing file example; |


#### G09 COM2 Command 

- `G09 COM2=XXXX` Send commands to another controller (for example 485) 
  
  Example: `G09 COM2=G00 J1=10 J2=10 J3=-90 J4=0 J5=0 J6=0`

- **Serial Port Communication:**
  - `G09 COM2 10H`: Sends `0X10` via Serial port 2.
  - `G09 COM2 12H`: Sends `0X12` via Serial port 2.
  - `G09 COM2 13H`: Sends `0X13` via Serial port 2.
  - `G09 COM2 14H`: Sends `0X14` via Serial port 2.
  - `G09 COM2 15H`: Sends `0X15` via Serial port 2.
  - `G09 COM2 18H`: Sends `0X18` via Serial port 2.

- **Special Instructions:** `G09 ENC....` is a special instruction for connecting with absolute value servo motors. Refer to the homing file example for detailed usage.

### VXX Unit
| Variable | Description |
|----------|-------------|
| VXX      | Unit        |
| #XX      | Immediate value |
| V0-V127, V400-V511 | User operation units |
| V144-V150 | System-used operation unit for input port values |
| V160-V166 | System-used operation unit for output port values |
| V176-V179 | System clock values |
| V180     | Real-time button value storage |
| V183     | Button value memory |
| V181     | Real-time button value storage |
| V184     | Button value memory |
| V182     | Real-time button value storage |
| V185     | Button value memory |
| V186     | Input i03 or i04 command cancel switch: 0 when not enabled, 1 when enabled |
| V187     | Limit and servo alarm master switch: 1 when not enabled, 0 when enabled |
| V188     | Soft limit stop master switch: 1 when not enabled, 0 when enabled |
| V189     | Input signal capture master switch: 1 when not enabled, 0 when enabled |
| V143     | Running pulse width (0~400, automatically modified) |
| V142     | Pulse width threshold (4~400, manually modified) |
| V141     | G41, G06 DGREE smoothing value (manually modified) |
| V140     | Speed timing value (automatically modified) |

### Corresponding Port Register Values

When using the scan command `G06 SCAN=I`:

- Input port values are stored in registers: `V144~V159, V192~V207`
- Register value is `0` when the light is on
- Register value is `1` when the light is off

| Input Ports | Register Values | Output Ports | Register Values | Homing Sensors | Register Values | Limit Switches | Register Values | Alarms | Register Values |
|-------------|-----------------|--------------|-----------------|----------------|-----------------|----------------|-----------------|--------|-----------------|
| i00:p0      | v144            | O00:P0       | v160            | HM0:P16        | v192            | LP0:P22        | v198            | ALM0:P28 | v204            |
| i01:p1      | v145            | O01:P1       | v161            | HM1:P17        | v193            | LP1:P23        | v199            | ALM1:P29 | v205            |
| i02:p2      | v146            | O02:P2       | v162            | HM2:P18        | v194            | LP2:P24        | v200            | ALM2:P30 | v206            |
| i03:p3      | v147            | O03:P3       | v163            | HM3:P19        | v195            | LP3:P25        | v201            | ALM3:P31 | v207            |
| i04:p4      | v148            | O04:P4       | v164            | HM4:P20        | v196            | LP4:P26        | v202            |                |                 |
| i05:p5      | v149            | O05:P5       | v165            | HM5:P21        | v197            | LP5:P27        | v203            |                |                 |
| i06:p6      | v150            | O06:P6       | v166            |                |                 |                |                 |                |                 |
| i07:p7      | v151            | O07:P7       | v167            |                |                 |                |                 |                |                 |
| i08:P8      | v152            | O08:P8       | v168            |                |                 |                |                 |                |                 |
| i09:P9      | v153            | O09:P9       | v169            |                |                 |                |                 |                |                 |
| i10:P10     | v154            | O10:P10      | v170            |                |                 |                |                 |                |                 |
| i11:P11     | v155            | O11:P11      | v171            |                |                 |                |                 |                |                 |
| i12:P12     | v156            | O12:P12      | v172            |                |                 |                |                 |                |                 |
| i13:P13     | v157            | O13:P13      | v173            |                |                 |                |                 |                |                 |
| i14:P14     | v158            | O14:P14      | v174            |                |                 |                |                 |                |                 |
| i15:P15     | v159            | O15:P15      | v175            |                |                 |                |                 |                |                 |


### Enable Capture (V256~V271: Value is 1 when capture enable is on, `V189=0` for it to take effect)
| Input Ports | Register Values |
|-------------|-----------------|
| ENi00:p0    | v256            |
| ENi01:p1    | v257            |
| ENi02:p2    | v258            |
| ENi03:p3    | v259            |
| ENi04:p4    | v260            |
| ENi05:p5    | v261            |
| ENi06:p6    | v262            |
| ENi07:p7    | v263            |
| ENi08:P8    | v264            |
| ENi09:P9    | v265            |
| ENi10:P10   | v266            |
| ENi11:P11   | v267            |
| ENi12:P12   | v268            |
| ENi13:P13   | v269            |
| ENi14:P14   | v270            |
| ENi15:P15   | v271            |

### Capture Duration (V288~V303: Duration value in milliseconds, needs software reset)
| Input Ports | Register Values |
|-------------|-----------------|
| FLAGi00:p0  | v288            |
| FLAGi01:p1  | v289            |
| FLAGi02:p2  | v290            |
| FLAGi03:p3  | v291            |
| FLAGi04:p4  | v292            |
| FLAGi05:p5  | v293            |
| FLAGi06:p6  | v294            |
| FLAGi07:p7  | v295            |
| FLAGi08:P8  | v296            |
| FLAGi09:P9  | v297            |
| FLAGi10:P10 | v298            |
| FLAGi11:P11 | v299            |
| FLAGi12:P12 | v300            |
| FLAGi13:P13 | v301            |
| FLAGi14:P14 | v302            |
| FLAGi15:P15 | v303            |


### Visual Example

#### Command Sequence (`G08 CCD2`)
This scenario illustrates communication and control between a Controller and an upper computer, where the Controller manages logic and sensor data while the PC sends positional commands based on certain conditions.

##### Interpretation:

The PLC code initializes and monitors V20, likely in response to some condition or trigger (T=401).
The upper computer responds with movement commands (G20 and G41), possibly based on visual or sensor inputs processed by the PLC (CCD2).
    
```
G08 CCD2:
G08 MOV V20=#0
G08 CCD2A:
G06 T=401  // RCCD
G08 IF V20==#0 AJMP CCD2A
G08 END
```
#### Upper Machine Command

When the upper machine receives T=401, it sends the following command:
```
G20 X=300 Y=200 Z=270 A=0 B=150 C=0 D=0 MOV V20=#1
G41 X=300 Y=200 Z=270 A=0 B=150 C=0 D=0 MOV V20=#1
```
### Input Signal Examples

#### 1. Initialization: Enable Activation (`ENi07:p7 v263`)

```
G08 CSH:
G08 MOV V189=#0
G08 MOV V263=#1
G08 END
```
#### 2. Reading Capture During Operation (`FLAGi07:p7 v295`)
```
G08 INSCAN:
G06 T=500
G08 IF V295>#400 ACALL ZHOUO14
G08 MOV V295=#0
G08 AJMP INSCAN
G08 END
```
#### 3. Capture Condition Greater than 400 Milliseconds Calls `ZHOUO14`
```
G08 ZHOUO14:
G06 O=P14.1
G06 T=400
G06 O=P14.0
G06 T=400
G06 O=P14.1
G06 T=400
G06 O=P14.0
G08 END
```
### Example: Implementation of for Loop in PFOP
#### C Language Example:
```
int V0;
int V1 = 100;
for (V0 = 2; V0 < V1; V0 = V0 + 3)
{
    // Loop body
    // ....
}
// Code after loop
// ....
```
#### Implementation in PFOP:
```
G08 MOV V0=#2   // Initialize V0
G08 MOV V1=#100 // Initialize V1

G08 AJMP FOR1   // Jump to the start of the loop

// Start of the loop body
// Loop body code
// ....

G08 ADD V0=V0+#3 // V0 += 3, update loop variable

// Check loop condition
G08 FOR1:
G08 IF_ELSE V0<V1?AJMP LABEL1:AJMP LABEL2 // If V0 < V1, jump to LABEL1 to continue looping; otherwise jump to LABEL2 to end loop

// Code continues after the loop
G08 LABEL2:
// Code after loop
// ....

```
### Example 2: Implementation of for Loop with break in PFOP
#### C Language Example:
```
int V0;
int V1 = 100;
for (V0 = 2; V0 < V1; V0 = V0 + 3)
{
    // Loop body
    // ....
    if (V0 > 50) {
        break;
    }
    // ....
}
// Code after loop
// ....
```
#### Implementation in PFOP:
```
G08 MOV V0=#2   // Initialize V0
G08 MOV V1=#100 // Initialize V1

G08 AJMP FOR1   // Jump to the start of the loop

G08 LABEL1:     // Start of the loop body
// Loop body code
// ....

G08 ADD V0=V0+#3 // V0 += 3, update loop variable

G08 IF V0>50 AJMP LABEL2 // Check break condition

// More loop body code
// ....

G08 FOR1:       // Check loop condition
G08 IF_ELSE V0<V1?AJMP LABEL1:AJMP LABEL2 // If V0 < V1, jump to LABEL1 to continue looping; otherwise jump to LABEL2 to end loop

G08 LABEL2:     // Code continues after the loop
// Code after loop
// ....
```
### Example 3: Implementation of `while` Loop in PFOP
#### C Language Example:
```
int V0 = 2;
int V1 = 100;
while (V0 < V1)
{
    V0 = V0 + 1;
    // Loop body
    // ....
}
// Code after loop
// ....
```
#### Implementation in PFOP:
```
G08 MOV V0=#2   // Initialize V0
G08 MOV V1=#100 // Initialize V1

G08 AJMP WHILE1 // Jump to the start of the loop

G08 WHILE1:     // Check loop condition
G08 IF V0<V1 ACALL LABEL1 // If V0 < V1, call LABEL1 to continue looping; otherwise jump to LABEL2 to end loop

// Loop body code
G08 ADD V0=V0+#1 // V0++, update loop variable

G08 AJMP WHILE1 // Continue looping

G08 LABEL1:     // Code continues after the loop
// Code after loop
// ....
```
### Example 3: Implementation of while Loop in PFOP
```
G08 MOV V0=#2   // Initialize V0
G08 MOV V1=#100 // Initialize V1

G08 AJMP WHILE1 // Jump to the start of the loop

G08 WHILE1:     // Check loop condition
G08 IF_ELSE V0<V1?AJMP LABEL1:AJMP LABEL2 // If V0 < V1, jump to LABEL1 to continue looping; otherwise jump to LABEL2 to end loop

// Loop body code
G08 ADD V0=V0+#1 // V0++, update loop variable

G08 AJMP WHILE1 // Continue looping

G08 LABEL1:     // Code continues after the loop
// Code after loop
// ....
```
### Example 4: Implementation of while Loop with break in PFOP
```
G08 MOV V0=#2   // Initialize V0
G08 MOV V1=#100 // Initialize V1

G08 AJMP WHILE1 // Jump to the start of the loop

G08 WHILE1:     // Check loop condition
G08 IF V0<V1 ACALL LABEL1 // If V0 < V1, call LABEL1 to continue looping; otherwise jump to LABEL2 to end loop

// Loop body code
G08 ADD V0=V0+#1 // V0++, update loop variable

G08 IF V0>50 AJMP LABEL2 // Check break condition

// More loop body code
// ....

G08 AJMP WHILE1 // Continue looping

G08 LABEL1:     // Code continues after the loop
// Code after loop
// ....
```
### Example 4: Implementation of `while` Loop with `break` in PFOP
```
G08 MOV V0=#2   // Initialize V0
G08 MOV V1=#100 // Initialize V1

G08 AJMP WHILE1 // Jump to the start of the loop

G08 WHILE1:     // Check loop condition
G08 IF V0<V1 ACALL LABEL1 // If V0 < V1, call LABEL1 to continue looping; otherwise jump to LABEL2 to end loop

// Loop body code
G08 ADD V0=V0+#1 // V0++, update loop variable

G08 IF V0>50 AJMP LABEL2 // Check break condition

// More loop body code
// ....

G08 AJMP WHILE1 // Continue looping

G08 LABEL1:     // Code continues after the loop
// Code after loop
// ....

G08 LABEL2:     // End of loop
// Loop termination code
// ....
```
```
FILE=INI
Parameter.ini
510

////DH_PARAMETER:
_h0=203;
_d1=80;
_h2=260;
_d3=80;
_h4=288;
_h5=64.5;
_d6=40;

////MOTOR_DIR:
_dir1=-1;
_dir2=1;
_dir3=1;
_dir4=-1;
_dir5=1;
_dir6=-1;

////JOINT_PUL:
_j1pul=320000;
_j2pul=326400;
_j3pul=326400;
_j4pul=320000;
_j5pul=320000;
_j6pul=320000;

////VELOCITY:
_ac=100000.0;
_de=100000.0;
_vpp=100000.0;
_vp=10;

////MODE:
_pve=8.887;
_getCodeMode=0;
_runCodeMode=1;
_prmTPUL=120;
_prmVLIM=2000;

//REPOS
_rePosJ1=0;
_rePosJ2=0;
_rePosJ3=-90;
_rePosJ4=0;
_rePosJ5=-90;
_rePosJ6=0;

//IRQ
_iIRQ=1;
_sIRQ=1;
_flagSoftRst=1;
_brkMotorEn=0;

//LIMIT
_sLp0=175;
_sLn0=-175;
_sLp1=90;
_sLn1=-90;
_sLp2=30;
_sLn2=-180;
_sLp3=175;
_sLn3=-175;
_sLp4=120;
_sLn4=-120;
_sLp5=359;
_sLn5=-359;
```

