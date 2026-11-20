# Raw Basics (without external libraries)
Below are all the basic operations of the **MPU6050 (gyro+acceleration module)** using **Arduino UNO R3**

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


