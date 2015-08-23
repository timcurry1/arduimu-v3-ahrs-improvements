# Introduction #

Add your content here.


# Details #

/**============================================================================================================
ARDUIMU V3

The**ARDUIMU V3**consists of the ATMEGA328 processor connected to a tri-axis HMC5883L Magnetometer
and a tri-axis MPU6000 Acceleromter/Angular Gyro. Note that these two sensors have onchip processors
and A/D converters which get the raw magnetometer, accelerometer and angular gyro data.  This differs
significantly from the previous IMUs (V1, V2) which used the ATMEGA328 A/Ds to get these sensor data.  The goods news
is this board now has 9 open digital and/or analog ports that you can use!  In the list below you will see those
designated by**Open**.  The MPU6000 has a processor which can be programmed to provide Euler angles and Quaternions
(have no idea what these are!) using something called the DMP (Dynnmic Motion Processor) which requires downloading
code into the device.  It seems to be a proprietry protocol to accomplish this but some people have done it. The
code presented here is rudimentary for both MPU6000 and HMC5883L - in their respective initialization code sets,
configuration registers are set which primarily sets the rate that the onchip processors will read the sensor
data and put them into registers.  Then when you read data from these devices, you are basically sending a register
address and reading the values.  For most parts you are reading two bytes and combining them for each sensor value.
There are no A/D conversions performed on the ATMEGA328.  Interestingly, both sensors are connected to interrupt
0 & 1 on the ATMEGA328 but they are not being used by the code.  The MPU6000 code does attach the interrupt and provides
an interrupt routine which increments a variable, but the variable is not being used in the code, but could be used
in the main processing loop to detect loops delays.  For the HMC5883L, although there is a physical connection to the
interrupt pin, the code does not invoke the interrupt.  If you really needed another I/O, you could cut trace and
make a physical connection tothis pin.**

The ARDUIMU V3 contain several ICs for 3.3V level conversion to interface to the sensors.  The board also contains
a 74157 Muliplexor which can cause a lot of head scratching.  This chip multiplexes the Serial Receive RX between
the USB FTDI chip and input from the GPS port.  On power up, it resets to get data from the USB port.  Therefore
when you are expecting data from the GPS, it must set high.  If your application is expecting input from your keyboard,
you must set this low.  When you are using GPS, the GPS library issues configuration commands to the GPS on the serial
line which will appear on the Serial monigtor output window at differ times which sometimes scrambles the serial output.
Just press reset button on the IMU.

Pin 13 is connected to the YELLOW LED, BUT it also connected to the SCK line of the MPU6000 - Strange - Do not try
to turn on Yellow LED, it will probably hose MPU6000 data.

Finally, the ATMEGA328 processor contains EEPROM which you can use to store calibration data during the calibration mode of
an application.  When in the running mode, the calibration data can be read from EEPROM. Speaking of calibration, both devices
must be calibrated to get any kind of reasonable data.  Mind you I am not a pro by any means on this topic but here is my
understanding of what calibration is all about.  For the accelerometer, the device puts out some positve steady state voltage
when it is a rest. So when you place it on a level surface, it may put out 1.5v say.  Now when you tilt it on an axis 90 degrees,
the reading will near 0v and when you tilt it to -90 degrees you would see 3v or so. So if you wanted to see your data go from
-1.5v to +1.5v for calculations later, then you would take the voltage from the level and subtract it from all your readings. So
this 1.5v value when the board is level is the calibration value, one for each of the XYZ axes.  In actuality we are not saving
the voltage value but the 16 bit integer value converted by the onboard processor which use it's A/Ds.  So we are essentially
reading this raw integer values from the designated registers of the device.  Since the data coming from the device is not rock
steady, in the calibration software routine, it is read many times, filtered, averaged, etc. to get a good value.  This calibrated
value if often called offsets/bias and is stored in EEPROM.  Now when the application is actually running, you get the values from
the device as you did in calibration, but now you subract the offset values to give you some signed binary (plus/minus) value for
use in your calculations.

For the angular rate gyro, things are a little different.  As it name implies, it outputs a voltage proportional to the rate at
which it is turning.  Thus if you were to put this device on a constantly rotating turntable you would see a contants voltage, say +1.0v.
Now if you reversed the rotation, you will see and -1.0v. Now with no rotation, if this was a perfect device, you should see 0.0v.
However, it isn't so you will have some value which could be near zero or not.  So like the accelerometer calibration, many readings
are taken to obtain this gyro calibration which will be a signed binary (plus/minus) value. As with the accelerometer, this calibration
value is use to correct subsequent reading from the device.

I am not that familiar with a magnetometer and not sure what references to hard & soft iron calibration means.  The device measures the
magnetic flux (hopefully of the earth) in it's XYZ axes.  Three problems (that I'm aware of) that crop up is that magnetic north is not the
same a geographic north (where Santa lives).  And to make things worse, it differs depending where you live AND in changes over the years.
I don't think we need to fret over the second one but the first we can fix by knowing the Magnetic Declination of where we live.  The second
problem is that it detects ANY kind of magnetic fields, so any wire with current flowing generates a magnetic field (think a quad copter with
four high current motors pulling varying current from the battery).  Remember to keep you board as far away possible from wires & motors.
And finally, the third is that magnetic fields get distorted by nearby ferrous materials (iron/steel), so it will distort the readings.
So calibration means you have to compensate your raw magnetometer readings for all these problems. This means you have to take readings
to get offsets like in the other sensors and you also have to get minimum and maximum readings from all three axes so you can adjust for
symmetry.  This assures that when you rotate on the XY plane, +90 degrees & -90 degrees rotations will yield correct results.


Some Relevant Links

ARDUIMU V3           https://code.google.com/p/ardu-imu/
ARDUIMU V3 Schematic http://dlnmh9ip6v2uc.cloudfront.net/datasheets/Robotics/ArduIMU328-v3.pdf
ATMEGA328            https://www.sparkfun.com/datasheets/Components/SMD/ATMega328.pdf

MPU6000 Acceleromter/Angular Gyro

MPU6000 3-Axis Accelerometer/Gyro - with onboard ADCs and processor

http://dlnmh9ip6v2uc.cloudfront.net/datasheets/Components/General%20IC/PS-MPU-6000A.pdf

Uses SPI Bus Interface
Access to MPU6000 is by reading from and writing to onboard registers.
Has an onboard processor and ADCs that continously reads the 3 axis gyro and accelerometer data
and puts all data in registers. The data read rate is configured using the MPUREG\_SMPLRT\_DIV
register set to 50hz ADCC rate. Configured to Interrupt the ATMega328 on Int0 Pin D2 when data is
ready and increments a byte MPU6000\_newdata - not being used in code.

HMC5883L 3-Axis Magnetometer

HMC5883L 3-Axis Magnetometer

http://dlnmh9ip6v2uc.cloudfront.net/datasheets/Sensors/Magneto/HMC5883L-FDS.pdf

I2C 1 Wire Interface
On-board processor to read all 3 magnetometer axis continuously
Reading data at 75hz, averaging 8 samples
Reads 6 bytes - 2 bytes for each axis

Chip has 4.7 kO pull-up resistors on the SDA line
and 2.2K resistor on the SCL line

Arduino Pin    HMC5883L Pin
D3 (INT1)      DRDY   Not Used!
A4 SDA         SDA
A5 SCL         SCL


ARDUIMU Pin Assignments/Mapping


> ATMEGA328      Arduino          ArduIMU Board
Pin#    Name      Name                 Name
1       D3         D3                DRDY to HMC5883L Magnetometer
2       D4         D4                FSYNC to MPU6000 Accel/Gyro - Device Select
3       GRD        Ground
4       VCC        5V
5       GRD        Ground
6       VCC        5V
7       XTAL       Crystal
8       XTAL       Crystal
9       D5         D5                Red LED
10      D6         D6                Blue LED
11      D7         D7                74157 Mux b/n FTDI & GPS Serial In
12      B0         D8                D8                    **Open**
13      B1         D9  PWM0          PWM0                  **Open**
14      B2         D10 PWM1          PWM1                  **Open**
15      B3         D11 MOSI          MOSI to MPU6000 Accel/Gyro
16      B4         D12 MISO          MISO to MPU6000 Accel/Gyro
17      B5         D13 SCK           SCK to MPU6000 Accel/Gyro - connected also to Yellow LED (do not use LED!)
18      AVCC       Analog VCC
19      A6         A6                A6                    **Open**
20      AREF       Analog Reference
21      AGRD       Analog Ground
22      A7         A7                A7                    **Open**
23      C0         A0                A0                    **Open**
24      C1         A1                A1                    **Open**
25      C2         A2                A2                    **Open**
26      C3         A3                A3                    **Open**
27      C4         A4 SDA            SDA to HMC5883L Magnetometer
28      C5         A5 SCL            SCL to HMC5883L Magnetometer
29      RESET      Chip Reset
30      D0         D0 RX-I           Serial RX from 74157 MUX - to FTDI & GPS port
31      D1         D1 TX-D           Serial TX to FTDI and GPS
32      D2         D2                Interrupt from MPU6000 Accel/Gyro

> GRD
> +5V - IN-OUT
> +VIN - 6 -12V
> +3.3V - OUT
> OUT - SERIAL
> IN - SERIAL
> RST - RESET
> +5V - OUT



Serial/FTDI Pins - 6 pin header
GRD      GRD
CTS      GRD
VCC      5V
RXI      TX from ARDUINO
TXO      RX to MUX D7 Selects input from FTDI chip or GPS connector
DTR      #RST - Reset



Sensor Orientation

X ROLL    along longer dimension of board
Y PITCH   along shorter dimension of board
Z YAW     vertical - down

XY REVERSE b/n ACCEL-GYRO & COMPASS
==============================================================================================================**/**





Add your content here.  Format your content with:
  * Text in **bold** or _italic_
  * Headings, paragraphs, and lists
  * Automatic links to other wiki pages