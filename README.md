#robotanno-commands
# RobotAnno Controller Command Set Documentation

#### 1. Serial Port Command Description

##### 1.1 Serial Port Mode
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

##### `0X05`: Query Mode
In any working mode of the control card (except file mode), sending 0X05 via the serial port can query the current working mode of the control card. The return values can be 0X10, 0X12, 0X13, 0X14, or 0X15.

##### `0X10`: Enter Idle Mode
When the control card is in idle mode (`0X10`), it remains in idle mode.

- In file mode, it takes effect after the file operation command is completed and `0X10` is sent.
- In homing mode, it takes effect after the robot stops moving (send `0X30`) and `0X10` is sent.
- In run mode, it takes effect after the robot stops moving (send `0X30`) and 0X10 is sent.
- In reset mode, it exits to idle mode after the reset is completed. It starts up in idle mode.

##### `0X11`: Enter File Mode
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

##### `0X12`: Motor Homing Mode
After entering homing mode, the system automatically runs the contents of the file and commands sent via the serial port in real time. Pause is 0X30, continue is 0X12, and exit is 0X10.
Instructions can be obtained from the run file or received via the serial port.
After running the file content, it automatically exits to idle mode.

##### `0X13`: Enter Run Mode
After entering run mode, the system automatically runs the contents of the file and commands sent via the serial port in real time. Pause is 0X30, continue is 0X13, and exit is 0X10.
Instructions can be obtained from the run file or received via the serial port.
After running the file content, it automatically starts running from the beginning.

##### `0X14`: Enter Debug Mode
Send 0X05 for query mode.

Send 0X10 to exit mode.

Send `0` to stop and send machine axis angles.

Send `SVON=1` to enable servo.

Send `SVON=0` to disable servo.

Send `SVBM=1` to release servo brake.

Send `SVBM=0` to engage servo brake.

Send `CLR=ST_PUL` to clear position on the control card.

Send `CLR=SV_ALM` to clear drive alarms.

Send `J#+` to move axis # in positive direction (# = 1, 2, 3, 4, 5, 6).

Send `J#-` to move axis # in negative direction.

Send `J#0` to stop axis # motion.

Send `J#<` to repeat motion for axis # (@ 0° to -90°).

Send `J#>` to repeat motion for axis # (@ 0° to 90°).

Send `P VE=####` to change running speed to #### pulses per second.

Send `P AC=####` to change acceleration to #### pulses per square second.

Send `P DE=####` to change deceleration to #### pulses per square second.

Send `G00 J#=####,....` to move robot joints to specified angles.

Send `G07 **=####` to change specified robot parameters.

TX+: Cartesian coordinate X+;

TX-: Cartesian coordinate X-;

TY+: Cartesian coordinate Y+;

TY-: Cartesian coordinate Y-;

TZ+: Cartesian coordinate Z+;

TZ-: Cartesian coordinate Z-;

TA+: Cartesian coordinate A+;

TA-: Cartesian coordinate A-;

TB+: Cartesian coordinate B+;

TB-: Cartesian coordinate B-;

TC+: Cartesian coordinate C+;

TC-: Cartesian coordinate C-;

P CYCP1=800000: J1 axis subdivided into 800000 pulses per revolution
....
P J1=24000: Assigning value to J1
....
P=VE: Query VE value. Available queries include VE, AC, DE, CYCP1, ..., J1, ..., H0, D1, H2, ...

##### `0X15`: Enter Reset Mode
Return the machine to the zero position.

### 2. Button Control Instructions

- On startup, pressing the `Start` button puts the robot into run mode.
- In run mode:
  - While the robot is running, pressing the `Stop` button pauses the robot.
  - When the robot is stopped, pressing the `Start` button resumes its operation.
  - When the robot is stopped, holding down the `Stop` button and then pressing the `Start` button will reset the robot and exit to idle mode.

## Instruction details:
```
G0n=GGN, such as G06 O=P1.1 is equivalent to GG6 O=P1.1
Point-to-point commands G00, G20
G00 J1=0 J2=0 J3=-90 J4=0 J5=-90 J6=0 // Absolute joint position command with acceleration and deceleration
G00 J1=VXX J2=0 J3=VXX J4=VXX J5=-90 J6=0 // Absolute joint position command with acceleration and deceleration
G00 ... J2'4 J3'VXX ... // Relative joint position command with acceleration and deceleration
G00 ... J2'4 J3'VXX J4=VXX J5=-90... // J2 is
G20 X=300 Y=100 Z=500 A=0 B=180 C=0 D=0 // With acceleration and deceleration
G20 X=300 Y=VXX Z=VXX A=VXX B=180 C=VXX D=0 // With acceleration and deceleration
G30 EQ01 J1=UXXXX,J2=UXXXX,J3=UXXXX Read from memory G30 EQ01 J1=U30100 J2=U30120
J3=U30140 J4=U30160
G30 EQ02 U3060=XX.XX Write to memory G30 EQ02 U30500=10
G30 EQ03 ASK=UXXXX Query memory content
G30 EQ04 Vxx=UXXXXXX
G30 EQ05 UXXXXXX=Vxx
G40 X=300 Y=100 Z=500 A=0 B=180 C=0 D=0 // Without acceleration and deceleration
```
```
Linear command G01 G21
G01 J1=0 J2=0 J3=-90 J4=0 J5=-90 J6=0
G21 X=300 Y=100 Z=500 A=0 B=180 C=0 D=0
G21 X=VXX Y=VXX Z=500 A=0 B=VXX C=0 D=0
G41 X=300 Y=100 Z=500 A=0 B=180 C=0 D=0 // With acceleration and deceleration
G41 X=300 Y=VXX Z=VXX A=0 B=180 C=0 D=0 // With acceleration and deceleration
```
```
Arc command G02 G03 G04 G22 G23 G06 DEGREE=ARC (or degree)
When DEGREE=ARC, it walks along a three-point arc
Current point: G21 X=200 Y=0 Z=200 A=-180 B=150 C=0 D=0
Second point: G22 X=300 Y=100 Z=200 A=-180 B=150 C=90 D=0
Third point: G23 X=400 Y=0 Z=200 A=-180 B=150 C=300 D=0
Arc operation: G06 DEGREE=300
```
```
G06 Command:
G06 T=500 Delay 500 milliseconds
G06 t=VXX Delay VXX milliseconds (e.g., G08 MOV V400=#2000 G06 t=V400 for 2000 milliseconds delay)
G06 I=P1.1 Wait for P1 to be high
G06 I=P2.0 Wait for P2 to be low
G06 O=P1.1 Set output port P1 to high (different port from input)
G06 O=P2.0 Set output port P2 to low
G06 SCAN=I Read input port values, store in V144-V150 cells
G06 SCAN=O Read output port values, store in V160-V166 cells
G06 SCAN=RTC Read system clock values, store in V176-V179 cells
G06 DEGREE=ARC Robot walks a three-point arc
G06 DEGREE=35.2 Robot walks 35.2 degrees
G06 REPOS=J# J# axis finds HOME# sensor in increasing angle direction (returns to low level)
G06 REPOS=-J# J# axis finds HOME# sensor in decreasing angle direction (returns to low level)
G06 REPOS=J1 J1 axis finds HOME0 sensor in increasing angle direction (returns to low level)
G06 REPOS=-J1 J1 axis finds HOME0 sensor in decreasing angle direction (returns to low level)
G06 REPOS=JH# J# axis finds HOME# sensor in increasing angle direction (returns to high level)
G06 REPOS=-JH# J# axis finds HOME# sensor in decreasing angle direction (returns to high level)
G06 REPOS=JH1 J1 axis finds HOME0 sensor in increasing angle direction (returns to high level)
G06 REPOS=-JH1 J1 axis finds HOME0 sensor in decreasing angle direction (returns to high level)
```
```
G07 指令
G07 VE=250000 速是 250000 脉冲每秒
G07 AC=250000 加速度是 250000 脉冲每秒平方
G07 DE=250000 减速度是 250000 脉冲每秒平方
G07 VPP=250000 最高速度 250000 脉冲每秒
G07 VP=20 速度 VE 是最高速度的 20%，VE=VPP*VP*0.01
G07 vp=VXX 速度 VE 是最高速度的 VXX 里的内容，VE=VPP*VXX 单元里的值*0.01
G07 _h0=xx 高度改变
G07 RCM=1 打印运行指令，0 不打印运行指令
G07 GCM=1 示教时以直角坐标输出，0 为关节坐标输出
G07 MARKPOS_HERE X1=0 Y1=0 X2=100 Y2=100 模板 MARK 点
G07 MARKPOS X1=10 Y1=10 X2=110 Y2=110 匹配 MARK 点
G07 UNIT=1.0 插补精度（单位 mm）
G07 P J#=XXXX 标定 J#轴角度
```
```
G08 Command:
G08 XXXX: Label (Labels cannot be followed by comments, label length must be less than 15 bytes)
G08 ACALL XXXX Call XXXX label until G08 END
G08 END End of call
G08 AJMP XXXX Jump to XXXX and execute
G08 FLTAB=# Jump to file T_# and execute
G08 IF VXX>=VXX ACALL XXXX Integer comparison immediately for # comparison >=, <=, ==, !=, >, < (G08 IF VXX>=VXX AJMP XXXX)
G08 IF_ELSE VXX>=VXX? ACALL XXXX: ACALL XXXX True value runs before colon, false value runs after colon ACALL is call, AJMP is jump (G08 IF_ELSE VXX>=VXX? AMMP XXXX: AJMP XXXX)
G08 IFF Floating point comparison
G08 IF_ELSEF Floating point comparison
G08 MOV VXX=#XX Integer transfer (immediate number G08 MOV V10=#12) (unit G08 MOV V10=V12)
G08 MOVF VXX=#XX Integer transfer (immediate number G08 MOVF V10=#12.5) (unit G08 MOVF V10=V12)
G08 PRINT VXX Print the value of VXX in integer form
G08 PRINTF VXX Print the value of VXX in floating point form
G08 INT VXX Convert floating point to integer ***************************
G08 FLOAT VXX Convert integer to floating point ************************
G08 ADD VXX=VXX+VXX Integer immediate number #
G08 SUBB VXX=VXX-VXX Integer immediate number #
G08 MUL VXX=VXX*VXX Integer immediate number #
G08 DIV VXX=VXX/VXX Integer immediate number #
G08 SQRT VXX=VXX Integer immediate number #
G08 ADDF VXX=VXX+VXX Floating point immediate number #
G08 SUBBF VXX=VXX-VXX Floating point immediate number #
G08 MULF VXX=VXX*VXX Floating point immediate number #
G08 DIVF VXX=VXX/VXX Floating point immediate number #
G08 SQRTF VXX=VXX Floating point immediate number #
G08 STO Program automatically pauses *************170817*************
G08 EXIT Exit running and enter idle mode *************170817*************
```
```
G09 Command:
G09 COPYRIGHT Ask system YN
G09 COM2=XXXX //COM2 port sends XXXX characters
Example: G09 COM2=G00 J1=10 J2=10 J3=-90 J4=0 J5=0 J6=0
//This controller sends commands to another controller COM1,
G09 COM2 10H //Serial port 2 sends 0X10
G09 COM2 12H //Serial port 2 sends 0X12
G09 COM2 13H //Serial port 2 sends 0X13
G09 COM2 14H //Serial port 2 sends 0X14
G09 COM2 15H //Serial port 2 sends 0X15
G09 COM2 18H //Serial port 2 sends 0X18
G09 ENC.... //Special instruction for connecting with absolute value servo motors, refer to the homing file example;
```

### G09 COM2 Command Description

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

```
VXX: Unit
#XX: Immediate value
User operation unit V0-V127, V400-V511
System-used operation unit reads input port values, stored in units V144-V150
Reads output port values, stored in units V160-V166
Reads system clock values, stored in units V176-V179
When running or running G-code, sending '0' stops, continuing is this instruction; '.' is also stop, continue is next instruction;
STA Real-time button value is stored in real time in unit V180, button value memory is stored in unit V183, cleared by software
STO Real-time button value is stored in real time in unit V181, button value memory is stored in unit V184, cleared by software
RST Real-time button value is stored in real time in unit V182, button value memory is stored in unit V185, cleared by software
V186: Input i03 or i04 as command cancel switch, V186=0 when not enabled (default), V186=1 when enabled
V187: Limit and servo alarm master switch, V187=1 when not enabled (default), V187=0 when enabled
V188: Soft limit stop master switch, V188=1 when not enabled (default), V188=0 when enabled
V189: Input signal capture master switch, V189=1 when not enabled (default), V189=0 when enabled
V143: Running pulse width (value range 0~400, automatically modified)
V142: Pulse width threshold (value range 4~400, manually modified)
V141: G41, G06 DGREE smoothing value (stop speed, manually modified)
V140: Speed timing value (automatically modified)
```
### Port Corresponding Register Values

| Port       | Corresponding Register Value |
|------------|------------------------------|
| **Input Ports (i00 to i15)** |
| i00        | p0 v144                      |
| i01        | p1 v145                      |
| i02        | p2 v146                      |
| i03        | p3 v147                      |
| i04        | p4 v148                      |
| i05        | p5 v149                      |
| i06        | p6 v150                      |
| i07        | p7 v151                      |
| i08        | P8 v152                      |
| i09        | P9 v153                      |
| i10        | P10 v154                     |
| i11        | P11 v155                     |
| i12        | P12 v156                     |
| i13        | P13 v157                     |
| i14        | P14 v158                     |
| i15        | P15 v159                     |
| **Home Sensors (HM0 to HM5)** |
| HM0        | P16 v192                     |
| HM1        | P17 v193                     |
| HM2        | P18 v194                     |
| HM3        | P19 v195                     |
| HM4        | P20 v196                     |
| HM5        | P21 v197                     |
| **Limit Switches (LP0 to LP5)** |
| LP0        | P22 v198                     |
| LP1        | P23 v199                     |
| LP2        | P24 v200                     |
| LP3        | P25 v201                     |
| LP4        | P26 v202                     |
| LP5        | P27 v203                     |
| **Alarm Signals (ALM0 to ALM3)** |
| ALM0       | P28 v204                     |
| ALM1       | P29 v205                     |
| ALM2       | P30 v206                     |
| ALM3       | P31 v207                     |
| **Output Ports (O00 to O15)** |
| O00        | P0 v160                      |
| O01        | P1 v161                      |
| O02        | P2 v162                      |
| O03        | P3 v163                      |
| O04        | P4 v164                      |
| O05        | P5 v165                      |
| O06        | P6 v166                      |
| O07        | P7 v167                      |
| O08        | P8 v168                      |
| O09        | P9 v169                      |
| O10        | P10 v170                     |
| O11        | P11 v171                     |
| O12        | P12 v172                     |
| O13        | P13 v173                     |
| O14        | P14 v174                     |
| O15        | P15 v175                     |

| Capture Enable Registers |
|--------------------------|
| **Capture Enable for Input Ports (ENi00 to ENi15)** |
| ENi00 | p0 v256 |
| ENi01 | p1 v257 |
| ENi02 | p2 v258 |
| ENi03 | p3 v259 |
| ENi04 | p4 v260 |
| ENi05 | p5 v261 |
| ENi06 | p6 v262 |
| ENi07 | p7 v263 |
| ENi08 | P8 v264 |
| ENi09 | P9 v265 |
| ENi10 | P10 v266 |
| ENi11 | P11 v267 |
| ENi12 | P12 v268 |
| ENi13 | P13 v269 |
| ENi14 | P14 v270 |
| ENi15 | P15 v271 |

| Capture Duration Flags |
|------------------------|
| **Capture Duration for Input Ports (FLAGi00 to FLAGi15)** |
| FLAGi00 | p0 v288 |
| FLAGi01 | p1 v289 |
| FLAGi02 | p2 v290 |
| FLAGi03 | p3 v291 |
| FLAGi04 | p4 v292 |
| FLAGi05 | p5 v293 |
| FLAGi06 | p6 v294 |
| FLAGi07 | p7 v295 |
| FLAGi08 | P8 v296 |
| FLAGi09 | P9 v297 |
| FLAGi10 | P10 v298 |
| FLAGi11 | P11 v299 |
| FLAGi12 | P12 v300 |
| FLAGi13 | P13 v301 |
| FLAGi14 | P14 v302 |
| FLAGi15 | P15 v303 |

### Visual Example

#### Command Sequence (`G08 CCD2`)

`plaintext`
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

`plaintext`
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

G08 LABEL1:     // Start of the loop body
// Loop body code
// ....

G08 ADD V0=V0+#3 // V0 += 3, update loop variable

G08 FOR1:       // Check loop condition
G08 IF_ELSE V0<V1?AJMP LABEL1:AJMP LABEL2 // If V0 < V1, jump to LABEL1 to continue looping; otherwise jump to LABEL2 to end loop

G08 LABEL2:     // Code continues after the loop
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

