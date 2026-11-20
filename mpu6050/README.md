# Raw Basics (without external libraries)
Below are all the basic operations of the **MPU6050 (gyro+acceleration module)** using **Arduino UNO R3**

**Mental model cheat:**
  - `beginTransmission()` → Start addressing the device.
  - `write()` → Specify what you want.
  - `endTransmission()` → Send that instruction.
  - `requestFrom()` → Request data from the device.


## 1. Simple Wake-up probing
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

## 2. Reading the values from registers
In this step we'll read through the values from its registers.

The process flows like below image:
<img width="768" height="388" alt="image" src="https://github.com/user-attachments/assets/556ea55e-a396-4e4b-ab9d-5cace49437b7" />

Steps:
  1. Start an I²C transmission to MPU6050_ADDR.
  2. Select the starting register (i.e.`0x3B`) using `Wire.write()`
  3. End the transmission.
  4. Request 6 bytes from the MPU6050 using `Wire.requestFrom()`

These are the **6 consecutive bytes** of MPU6050's Accelerometer:
| Register address | Register name | What it stores |
| :--- | :--- | :--- |
| `0x3B` | ACCEL_XOUT_H | Accelerometer X high byte |
| `0x3C` | ACCEL_XOUT_L | Accelerometer X low byte |
| `0x3D` | ACCEL_YOUT_H | Accelerometer Y high byte |
| `0x3E` | ACCEL_YOUT_L | Accelerometer Y low byte |
| `0x3F` | ACCEL_ZOUT_H | Accelerometer Z high byte |
| `0x40` | ACCEL_ZOUT_L | Accelerometer Z low byte |

And **6 consecutive bytes** of MPU6050's Gyroscope:
| Register address | Register name | What it stores |
| :--- | :--- | :--- |
| `0x43` | GYRO_XOUT_H | Gyroscope X high byte |
| `0x44` | GYRO_XOUT_L | Gyroscope X low byte |
| `0x45` | GYRO_YOUT_H | Gyroscope Y high byte |
| `0x46` | GYRO_YOUT_L | Gyroscope Y low byte |
| `0x47` | GYRO_ZOUT_H | Gyroscope Z high byte |
| `0x48` | GYRO_ZOUT_L | Gyroscope Z low byte |

__To read the data we need to read the 6 consecutive bytes via `.requestFrom()` starting from `0x3B` in case of Accelerometer & `0x43` for Gyro__

Code for accelerometer:
```cpp
#include <Wire.h>

#define MPU6050_ADDR 0x68

void setup() {
  Serial.begin(115200);
  Wire.begin();  

}

void loop() {
  Wire.beginTransmission(MPU6050_ADDR);
  Wire.write(0x3B); // setting the stating byte point to read (`0x3B` to `0x40`)
  byte error = Wire.endTransmission();

  if(error != 0) Serial.println("Connection failed!");

  Wire.requestFrom(MPU6050_ADDR, 6); // requesting to read all 6 consecutive bytes starting from the initial `0x3B` we set above

  if(Wire.available() == 6) Serial.println("Received all 6 bytes!"); // checks if we have the all the 6 bytes data(unread) available, on i2c buffer

  delay(1000);
}

```

Now we need to actually read and print out the values

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
  Wire.write(0x3B);
  byte error = Wire.endTransmission();

  if (error != 0) Serial.println("Connection failed!");

  Wire.requestFrom(MPU6050_ADDR, 6);

  if(Wire.available() == 6){
    byte accX_H = Wire.read();
    byte accX_L = Wire.read();

    byte accY_H = Wire.read();
    byte accY_L = Wire.read();

    byte accZ_H = Wire.read();
    byte accZ_L = Wire.read();

    Serial.print("X high:");
    Serial.println(accX_H);
    Serial.print("X low:");
    Serial.println(accX_L);

    Serial.print("Y high:");
    Serial.println(accY_H);
    Serial.print("Y low:");
    Serial.println(accY_L);

    Serial.print("Z high:");
    Serial.println(accZ_H);
    Serial.print("Z low:");
    Serial.println(accZ_L);
  }

  delay(500);
}

```

