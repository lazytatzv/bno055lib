# bno055lib

A robust, thread-safe, and dependency-free C++17 library for the BNO055 sensor over I2C on Linux.

Linux用のI2C経由BNO055制御C++17ライブラリ。

---

## Build & Install

```bash
mkdir build && cd build
cmake ..
make
sudo make install
```

---

## API Reference

### Namespaces & Types

```cpp
namespace bno055lib {
    struct Vector3 { double x, y, z; };
    struct Quaternion { double w, x, y, z; };
    
    struct Offsets {
        int16_t accel_offset_x, accel_offset_y, accel_offset_z;
        int16_t mag_offset_x, mag_offset_y, mag_offset_z;
        int16_t gyro_offset_x, gyro_offset_y, gyro_offset_z;
        int16_t accel_radius, mag_radius;
    };

    struct CalibrationStatus {
        uint8_t sys;   // 0 to 3
        uint8_t gyro;
        uint8_t accel;
        uint8_t mag;
        bool isFullyCalibrated() const;
    };

    enum class OpMode : uint8_t {
        Config = 0X00, AccOnly = 0X01, MagOnly = 0X02, GyroOnly = 0X03,
        AccMag = 0X04, AccGyro = 0X05, MagGyro = 0X06, AMG = 0X07,
        IMUPlus = 0X08, Compass = 0X09, M4G = 0X0A, NDOF_FMC_Off = 0X0B,
        NDOF = 0X0C
    };

    enum class LogLevel { Debug, Info, Warning, Error };
    using LoggerCallback = std::function<void(LogLevel level, std::string_view message)>;
}
```

### Class `BNO055`

#### Lifecycle
* **`explicit BNO055(uint8_t i2c_address = 0x28, std::string_view i2c_device = "/dev/i2c-1")`**
* **`bool begin(OpMode mode = OpMode::NDOF)`**

#### Configuration
* **`void setMode(OpMode mode)`**
* **`OpMode getMode()`**
* **`void setAxisRemap(AxisMapConfig config)`**
* **`void setAxisSign(AxisMapSign sign)`**
* **`void setExtCrystalUse(bool use_xtal)`**

#### Sensor Data (SI Units)
* **`Vector3 getAccelerometer()`** (returns $m/s^2$)
* **`Vector3 getMagnetometer()`** (returns $\mu T$)
* **`Vector3 getGyroscope()`** (returns $rad/s$)
* **`Vector3 getEulerAngles()`** (returns `x = Roll`, `y = Pitch`, `z = Yaw` in $rad$)
* **`Vector3 getLinearAcceleration()`** (returns $m/s^2$)
* **`Vector3 getGravity()`** (returns $m/s^2$)
* **`Quaternion getQuaternion()`**
* **`int8_t getTemperature()`** (returns Celsius)

#### Calibration & Offsets
* **`CalibrationStatus getCalibrationStatus()`**
* **`bool getSensorOffsets(Offsets& offsets)`**
* **`bool getSensorOffsets(std::array<uint8_t, 22>& calib_data)`**
* **`void setSensorOffsets(const Offsets& offsets)`**
* **`void setSensorOffsets(const std::array<uint8_t, 22>& calib_data)`**
* **`bool saveCalibrationFile(std::string_view filepath)`**
* **`bool loadCalibrationFile(std::string_view filepath)`**

#### Power Management
* **`void enterSuspendMode()`**
* **`void enterNormalMode()`**

#### Logging
* **`void setLogger(LoggerCallback callback)`**

---

## License

MIT License.
