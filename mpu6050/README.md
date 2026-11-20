# Raw Basics (without external libraries)
Below are all the basic operations of the **MPU6050 (gyro+acceleration module)** using **Arduino UNO R3**

**Mental model cheat:**
  - `beginTransmission()` → Start addressing the device.
  - `write()` → Specify what you want.
  - `endTransmission()` → Send that instruction.
  - `requestFrom()` → Request data from the device.


## Simple Wake-up probing
Operation Steps:
  1. prepare the arduino for i2c comm with `Wire.begin()`
  2. init the transmission sequence with `Wire.beginTransmission()`
  3. write `0x00` into mpu's power management port to disable sleep mode & wake it up
  4. finally send the write data to the mpu via `Wire.endTransmission()`



```cpp
#include <Wire.h> // used for I2C comm

#define MPU6050_ADDR 0x68 // default ADDR for this module unless changed
#define PWR_MGMNT_1 0x6B // its default power management ADDR

void setup() {
  // put your setup code here, to run once:
  Serial.begin(115200);

  Wire.begin(); // begins the I2C communication between the arduino and the target I2C device (mpu6050 in this case)

  Wire.beginTransmission(MPU6050_ADDR); // prepares the module to send data to its corresponding addr
  Wire.write(PWR_MGMNT_1);
  Wire.write(0x00); // these two lines write value '0' to the pwr mngmnt addr where '0' disables the sleep mode and wakes it up
  byte error = Wire.endTransmission(); // actually sends the written data to the module [ Wire.endTransmission() returns ops status code]

  if(error == 0) Serial.println("MPU6050 is awake!");
  else Serial.println("Communication failed :(");


}

void loop() {
  // put your main code here, to run repeatedly:

}
```

## Reading the values from registers
In this step we'll read through the values from its registers.

The process flows like below image:
<img width="768" height="388" alt="image" src="https://github.com/user-attachments/assets/556ea55e-a396-4e4b-ab9d-5cace49437b7" />

Steps:
  1. Start an I²C transmission to MPU6050_ADDR.
  2. Select the starting register `0x3B` using `Wire.write()`
  3. End the transmission.
  4. Request 6 bytes from the MPU6050 using `Wire.requestFrom()`

These are the **6 consecutive bytes** of Accelerometer we're gonna read first:
| Register address | Register name | What it stores |
| :--- | :--- | :--- |
| `0x3B` | ACCEL_XOUT_H | Accelerometer X high byte |
| `0x3C` | ACCEL_XOUT_L | Accelerometer X low byte |
| `0x3D` | ACCEL_YOUT_H | Accelerometer Y high byte |
| `0x3E` | ACCEL_YOUT_L | Accelerometer Y low byte |
| `0x3F` | ACCEL_ZOUT_H | Accelerometer Z high byte |
| `0x40` | ACCEL_ZOUT_L | Accelerometer Z low byte |

Simpler visual:
| Axis | High byte | Low byte |
| :--- | :--- | :--- |
| X | `0x3B` | `0x3C` |
| Y | `0x3D` | `0x3E` |
| Z | `0x3F` | `0x40` |

So we want to read the 6 consecutive bytes via `.requestFrom()` starting from `0x3B`

Code:
```cpp
#include <Wire.h>

#define MPU6050_ADDR 0x68

void setup() {
  Serial.begin(115200);
  Wire.begin();  

}

void loop() {
  Wire.beginTransmission(MPU6050_ADDR);
  Wire.write(0x3B); // we're gonna read 6 consecutive bytes stating from mpu's `0x3B` (`0x3B` to `0x40`)
  byte error = Wire.endTransmission();

  if(error != 0) Serial.println("Connection failed!");

  Wire.requestFrom(MPU6050_ADDR, 6); // requesting to read all 6 consecutive bytes starting from the initial `0x3B` we set above

  if(Wire.available() == 6) Serial.println("Received all 6 bytes!"); // checks if we actually have the all the 6 bytes available on i2c buffer

  delay(1000);
}

```

