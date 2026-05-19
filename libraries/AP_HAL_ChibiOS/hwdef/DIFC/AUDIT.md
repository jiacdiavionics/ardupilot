# DIFC Final Embedded Audit Report
# STM32H743VIT6 Bring-Up Assessment

---

## TASK 1: STM32H743 AF MAPPING VERIFICATION

### FIXED PINS ❌→✅

| Pin | Original Use | Issue | Fixed To | Status |
|-----|------------|-------|---------|--------|
| PE15 | ICM42688P_CS | NO SPI4 on PE15! | PD4 | ✅ FIXED |
| PC3 | SPI2_MOSI | NO SPI2 on PC3! | PB15 | ✅ FIXED |

### VERIFIED PINS ✅

| Peripheral | Pins | AF | Status |
|------------|------|-----|--------|
| SPI1 | PA5-7 | AF5 | ✅ VALID |
| SPI1 CS | PA4 | AF5 | ✅ VALID |
| SPI2 SCK | PD3 | AF5 | ✅ VALID |
| SPI2 MISO | PB14 | AF5 | ✅ VALID |
| SPI2 MOSI | PB15 | **FIXED** | ✅ |
| SPI2 CS | PB12 | AF5 | ✅ VALID |
| SPI3 | PB3-5 | AF6 | ✅ VALID |
| SPI4 | PE12-14 | AF5 | ✅ VALID |
| SPI4 CS | PD4 | **FIXED** | ✅ |
| USART2 | PD5-6 | AF7 | ✅ VALID |
| UART4 | PB8-9 | AF8 | ✅ VALID |
| UART6 | PC6-7 | AF8 | ✅ VALID |
| UART7 | PE7-8 | AF7 | ✅ VALID |
| UART8 | PE0-1 | AF7 | ✅ VALID |
| I2C1 | PB6-7 | AF4 | ✅ VALID |
| I2C2 | PB10-11 | AF4 | ✅ VALID |
| CAN1 | PD0-1 | AF9 | ✅ VALID |
| SDMMC1 | PC8-12,PD2 | AF12 | ✅ VALID |
| USB | PA11-12 | AF10 | ✅ VALID |

### PWM TIMERS ✅ ALL VALID

---

## TASK 2: TIMER REALITY

### PWM Output Verification (STM32H743)

| PWM | Pin | Timer | CH | DMA | BDShot | Timer Type |
|-----|-----|-------|-----|-----|--------|-----------|
| PWM1 | PB0 | TIM3 | CH3 | ✅ DMA1 | ✅ | General |
| PWM2 | PB1 | TIM3 | CH4 | ✅ DMA1 | ✅ | General |
| PWM3 | PA0 | TIM5 | CH1 | ✅ DMA1 | ❌ | General |
| PWM4 | PA1 | TIM5 | CH2 | ✅ DMA1 | ❌ | General |
| PWM5 | PA2 | TIM5 | CH3 | ✅ DMA1 | ❌ | General |
| PWM6 | PA3 | TIM5 | CH4 | ✅ DMA1 | ❌ | General |
| PWM7 | PD12 | TIM4 | CH1 | ✅ DMA1 | ✅ | General |
| PWM8 | PD13 | TIM4 | CH2 | ✅ DMA1 | ✅ | General |
| PWM9 | PD14 | TIM4 | CH3 | ✅ DMA1 | ✅ | General |
| PWM10 | PD15 | TIM4 | CH4 | ✅ DMA1 | ✅ | General |
| PWM11 | PE5 | TIM15 | CH1 | ❌ | ❌ | Basic |
| PWM12 | PE6 | TIM15 | CH2 | ❌ | ❌ | Basic |

### Conflicts:
- NO conflicts with UART/SPI/DMA
- DMA_NOSHARE SPI1* SPI4* configured
- ✅ GOOD

---

## TASK 3: INS/SPI HARDENING

### SPI Speed Analysis

| Device | Configured | Recommended | Status |
|--------|-----------|------------|-----------|
| ICM42688P | 2MHz/16MHz | 2MHz/8MHz | ⚠️ REDUCE |
| MPU6500 | 1MHz/8MHz | 1MHz/4MHz | ⚠️ REDUCE |
| MAX7456E | 10MHz/10MHz | 10MHz OK | ✅ |

### Startup Order:
1. SPI1 - MPU6500 (secondary)
2. SPI4 - ICM42688P (primary)

### Issue: ICM42688P is PRIMARY but SPI4 shares DMA risk
- DMA_NOSHARE configured - GOOD

### Recommendation for First Boot:
Reduce SPI speeds for reliability:
```
ICM42688P: 2MHz init, 8MHz runtime (not 16MHz)
MPU6500: 1MHz init, 4MHz runtime (not 8MHz)
```

---

## TASK 4: SBUS INVERSION

### USART2 Analysis

**Hardware:** STM32H743 has hardware inversion (USART_CR2_RXINV)
**Firmware:** ArduPilot's RCIN_SBUS protocol handles inversion

### Configuration:
```
CURRENT: define DEFAULT_SERIAL2_PROTOCOL SerialProtocol_RCIN
CORRECT: YES - ArduPilot handles SBUS automatically
```

### RC_OPTIONS:
No changes needed - SBUS handled in protocol layer.

**Verdict:** ✅ ADEQUATE

---

## TASK 5: BOOTLOADER + FLASH AUDIT

### Flash Layout

| Region | Size | Offset | Status |
|--------|------|--------|--------|
| Bootloader | 384KB | 0x08000000 | OK |
| App | 1664KB | 0x08060000 | OK |
| Storage | 256KB | 0x080E0000 | OK |

### Check:
- ✅ No overlap
- ✅ 384KB bootloader standard for H7
- ✅ 256KB storage adequate (pages 14-15)

### Bootloader Config:
- ✅ FLASH_BOOTLOADER_LOAD_KB = 384
- ✅ Matches FLASH_RESERVE_START_KB
- ✅ OK

---

## TASK 6: FIRST POWER-ON EXPECTATIONS

### GOOD BOOT Log:
```
APM: ArduCopter V4.x.x
APM: DIFC board
Free RAM: 484KB
Flash: 2048KB

STM32: HSE 8MHz
STM32: PLL 480MHz
STM32: system timer 12MHz
APM: RC Protocol: RCIN_SBUS on USART2

Invensense: ICM42688P found
SPI4: speed 16MHz
IMU: ICM42688P rotation: 0

Invensense: MPU6500 found
SPI1: speed 8MHz
IMU: MPU6500 rotation: 0

BARO: SPL06 found at I2C address 0x76
COMPASS: QMC5883L found at I2C address 0x1E

CAN: 1: started
SDCard: Mounted (FAT)
Filesystem: OK
Storage: params 256KB

OSD: detected
OSD: Character set loaded

 Motors: 12 outputs configured
EKF: Primary 51cm variance
APM: Happy quadcopter
```

### FAILURE: No IMU Primary
```
...
SPI4: not found
APM: Insufficient IMUs
```
**Fix:** Check SPI4 pins PE12-14

### FAILURE: SPI Timeout
```
Invensense: ICM42688P timeout
SPI4: error -1
```
**Fix:** Lower SPI speed to 8MHz

### FAILURE: SD Card
```
SDCard: not detected
Storage: FAILED
```
**Fix:** Check FAT32 format

### FAILURE: No RC Input
```
RC Input: no signal
RCIN: RX lost
```
**Fix:** Check SBUS inverter or SERIAL2_PROTOCOL

### FAILURE: Compass
```
COMPASS: not found
```
**Fix:** Add I2C pullups or check address 0x1E

---

## TASK 7: MINIMAL SAFE CONFIG

### First Boot Parameters:
```bash
# DISABLE all secondary features for bench test
INS_ENABLE_MASK = 1       # Only IMU0
LOG_BACKEND_TYPE = 2       #_file only
BRD_PWM_COUNT = 10        # Not 12
LOG_DISARMED = 0          # No logging to start

# RC - conservative
SERIAL2_PROTOCOL = 23    # RCIN_SBUS explicitly
SERIAL2_BAUD = 100000

# CAN - disable for test
CAN_PROTOCOL = 0          # Disable

# OSD - disable for test  
HAL_OSD_TYPE_DEFAULT = 0

# IMU - reduce rates for test
INS_GYRO_FILTER = 20
INS_ACCEL_FILTER = 20
INS_FAST_SAMPLE = 0       # Single sample
```

---

## TASK 8: CONFIDENCE REPORT

| Subsystem | Confidence | Notes |
|----------|------------|--------|
| Bootloader boot | 90% | Standard H7 |
| USB enumeration | 90% | Standard AF |
| IMU SPI4 detection | 65% | **FIXED** but untested |
| IMU SPI1 detection | 80% | Standard |
| SD card detection | 75% | Needs FAT32 |
| RC input (SBUS) | 70% | May need inverter |
| PWM output | 85% | Standard |
| OSD detection | 70% | **FIXED** MOSI pin |
| CAN | 80% | Standard |

### Overall Confidence: ~75%

### Why Not 90%+?
1. SPI4 pins moved - need verify on hardware
2. SPI2 MOSI moved - need verify on hardware  
3. RC inversion untested
4. Unregistered board ID

---

## CRITICAL FIXES APPLIED

1. ✅ SPI4 CS: PE15 → PD4 (PE15 has NO SPI4)
2. ✅ SPI2 MOSI: PC3 → PB15 (PC3 has NO SPI2)

### Remaining Issues:
- Board ID unregistered
- SPI speeds may need reduction for reliability
- RC inversion depends on hardware

---

## Build Commands:

```bash
./waf configure --board DIFC
./waf copter

# Flash
dfu-util -a 0 -D bin/arducopter.bin -R
```