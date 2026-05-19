# DIFC Hardware Bring-Up Checklist
# Critical first-power sequence for STM32H743VIT6 bring-up

---

## PHASE 1: BOOTLOADER & DFU

### 1.1 DFU Detection
Expected: Board enters DFU mode on USB (0x1209:0x0002 or similar)

Check:
```bash
$ dfu-util -l
```
Expected output:
```
Found DFU: [0483:df11] ver=2200, devnum=13, cfg=1, intf=0, alt=0, 
name="@Data Flash@   /0x08000000/01*128Ke"
```

### 1.2 Flash Bootloader
Check flash layout:
```
0x08000000-0x08060000: Bootloader (384KB)
0x08060000-0x080FFFFF: Main firmware (1.6MB)
0x080E0000-0x080FFF00: Storage (pages 14-15)
```

Verify with:
```bash
$ stm32flash -S 0x08000000 /dev/ttyUSB0
```

### 1.3 USB Enumeration
Expected in dmesg after bootloader flash:
```
[  +0.000000] usb 1-1: new device
[  +0.000000] usb 1-1: device using address 5
```

---

## PHASE 2: BASIC FIRMWARE LOAD

### 2.1 First Boot Messages
Expected Mission Planner console:
```
APM: ArduCopter V4.x.x
APM: DIFC board
Free RAM: 484KB
Flash: 2048KB
```

### 2.2 System Timer
Expected:
```
STM32: system timer 12MHz
APM: RC Protocol: RCIN_SBUS on USART2
```

---

## PHASE 3: SENSOR PROBING

### 3.1 IMU Primary (ICM42688P on SPI4)
Expected messages:
```
Invensense: ICM42688P found
SPI4: speed 16MHz
IMU: ICM42688P rotation: 0
```

Failure symptom: "SPI4: not found" or timeout
Debug:
```
.param show INS_PROBE
```

### 3.2 IMU Secondary (MPU6500 on SPI1)
Expected:
```
Invensense: MPU6500 found
SPI1: speed 8MHz
IMU: MPU6500 rotation: 0
```

### 3.3 Barometer (SPL06 on I2C2)
Expected:
```
BARO: SPL06 found at I2C address 0x76
```

Frequent failure - try alternative I2C2 address:
```
BARO: SPL06 not found - try BMP280 on 0x76
```

### 3.4 Compass (QMC5883L on I2C1)
Expected:
```
COMPASS: QMC5883L found at I2C address 0x1E
```

---

## PHASE 4: SD CARD DETECTION

### 4.1 SDMMC Detection
Expected:
```
SDCard: Mounted
Filesystem: FATFS
Free: XXX KB
```

Failure: "SDCard: not detected"
Fix: Check SD card format (must be FAT32)

### 4.2 Storage
Expected:
```
Storage: pages 14-15
Parameters: OK
```

---

## PHASE 5: RC INPUT

### 5.1 SBUS on USART2
Expected:
```
RC Input: RCIN_SBUS
RSSI: via RC channel
```

### 5.2 RC Signal Valid
Check in Mission Planner:
- RC Input > 1000 when transmitting
- RC Input value should fluctuate

**CRITICAL: CHECK SBUS INVERSION**
SBUS is inverted signal - add to parameters:
```
SERIAL2_PROTOCOL = 23
```
or use inverted UART mode:

If no RC signal, check:
```bash
$ serial2_protocol
# Should show: 23 = RCIN_SBUS
```

---

## PHASE 6: PWM VALIDATION

### 6.1 PWM Outputs (Scope Check)
Test each output:
```bash
$ servo set 1 1500
$ servo set 2 1500
```

On oscilloscope:
- PWM1 (PB0): Expected 1.5ms @ 50Hz
- PWM2 (PB1): Expected 1.5ms @ 50Hz
- PWM3-PWM6: TIM5 outputs
- PWM7-PWM10: TIM4 outputs (BDShot capable)
- PWM11-PWM12: TIM15 outputs

### 6.2 BDShot Check
```bash
$ if (not found in params) motor/brig
# Check BRD_PWM_COUNT
# Try BDShot protocol on outputs 7-10
```

### 6.3 PWM Frequency
Default: 50Hz for PWM, 400Hz for H7 if running
Check:
```bash
$ PWM_OUTPUT_SKIP
```

---

## PHASE 7: IMU VIBRATION CHECKS

### 7.1 IMU Health
Expected vibration levels:
- Accel: < 50 mg/√Hz ideal
- Accel: < 200 mg/√Hz OK
- Gyro: < 10 mdps/√Hz ideal

Check in MP:
```
IMU0: 0.1 0.1 0.1
IMU1: 0.1 0.1 0.1
```

### 7.2 Filtering
Check:
- INS_GYRO_FILTER = 20 (default)
- INS_ACCEL_FILTER = 20

### 7.3 Notch Filter
If high vibration:
```bash
$ FFT_ENABLE = 1
$ FFT_MINHZ = 40
$ FFT_MAXHZ = 200
```

---

## PHASE 8: CAN VALIDATION

### 8.1 CAN Bus
Expected:
```
CAN: 1: started
```

### 8.2 CAN Devices
List CAN devices:
```bash
$ can show
```

### 8.3 CAN Termination
Bus must have 120Ω termination at both ends
- Test with Ohm meter across CANH-CANL
- Should read ~60Ω (two 120Ω in parallel)

---

## PHASE 9: OSD VALIDATION

### 9.1 OSD Detect
Expected on boot:
```
OSD: OSD detected
OSD: Character set loaded
```

### 9.2 OSD Test Screen
Check:
- Battery voltage visible
- RC RSSI visible
- Flight mode visible

---

## MAVPROXY COMMANDS

```bash
# Console monitoring
mavproxy.py --master=/dev/ttyUSB0 --baud=115200 --console

# Log download
mavproxy.py --master=/dev/ttyUSB0 --log

# Parameter save
mavproxy.py --master=/dev/ttyUSB0 --param save DIFC.parm

# Force IMU sampling
params set INS_GYRO_FILTER 20
params set INS_ACCEL_FILTER 20

# RC test
rc 1 1500

# Motor test
motor test 1
```

---

## COMMON FAILURE MODES

| Symptom | Likely Cause | Fix |
|---------|------------|------|
| No USB connect | Bootloader not flashed | Flash bootloader first |
| No IMU found | SPI4 AF wrong | Verify PE12-14 SPI4 |
| No barometer | I2C2 address wrong | Try 0x76, 0x77 |
| No compass | I2C1 not detected | Check I2C pullups |
| No RC input | SBUS not inverted | Set SERIAL2_PROTOCOL |
| No PWM output | Timer misconfigured | Check TIM3/4/5 mapping |
| No CAN | Termination missing | Add 120Ω at ends |
| No SD card | FAT32 required | Reformatter SD |
| High IMU noise | Vibration | Add vibration isolation |
| ADC reads 0 | Wrong pin config | Verify PC0/PC1 mapping |

---

## TROUBLESHOOTING COMMANDS

```bash
# Check all parameters
param show

# Show IMU status
ins show

# Reset to factory
param default

# Check free memory
free

# Show USB status
usb start

# Check system timer
system time

# IMU raw
raw IMU

# Serial list
serial
```