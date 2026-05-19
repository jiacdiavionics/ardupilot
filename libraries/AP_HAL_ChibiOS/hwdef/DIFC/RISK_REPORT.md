# DIFC Board Port - Critical Risk Report
# Embedded bring-up risk assessment

---

## HIGH RISK ITEMS ⚠️

### 1. USART2 SBUS Inversion (CRITICAL)
**Issue:** No SBUS inversion flag set. SBUS is inverted TTL signal.
**Impact:** RC input will not work on first boot.
**Fix:** Must set `SERIAL2_PROTOCOL = 23` (RCIN_SBUS) which handles inversion OR add NODMA + inversion flags to USART2.
**Timeline:** First power-on will fail RC.

### 2. Board ID Not Registered (CRITICAL)  
**Issue:** `AP_HW_DIFC` must be registered in ArduPilot for bootloader.
**Impact:** Build may fail; DFU won't recognize board.
**Fix:** Obtain unique ID from maintainers before production.

### 3. SPI4 Alternate Functions Unverified (HIGH)
**Issue:** PE12/PE13/PE14 selected for SPI4 but need manual verification against STM32H743 datasheet.
**Impact:** ICM42688P will not communicate.
**Fix:** Verify pins PE12-14 have SPI4 AF in STM32H743. May need different pins.

### 4. ADC Pin Mapping May Be Wrong (HIGH)
**Issue:** Line 161 uses HAL_BATT_VOLT_PIN=10 but format unclear.
**Impact:** Battery monitoring shows 0 or wrong values.
**Fix:** Verify PC0 channels correctly. Try "PC0" or index format.

---

## MEDIUM RISK ITEMS ⚠️

### 5. TIM15 No DMA - PWM11/PWM12 Latency
**Issue:** TIM15 has no DMA capability on STM32H743.
**Impact:** High latency on outputs 11-12, unsuitable for fast servos.
**Fix:** Don't use PWM11-12 for critical functions.

### 6. No SD Card Detect GPIO
**Issue:** No GPIO defined for SD card insert detection.
**Impact:** Can't detect SD removal/insertion.
**Fix:** Add GPIO for card detect if needed.

### 7. Missing HAL_WITH_MCU_MONITORING
**Issue:** No UART assigned for MCU health monitoring.
**Impact:** Can't see CPU temperature, faults.
**Fix:** Can add later if needed.

### 8. No crashdump Configuration
**Issue:** No CORTEX_FPU or crashdump defines.
**Impact:** Harder to debug crashes.
**Fix:** Add defines for production.

### 9. WS2812 LED Not Enabled  
**Issue:** Potential PA8/TIM1 conflict with Buzzer.
**Impact:** RGB LED can't be used.
**Fix:** Enable later if board has RGB.

### 10. I2C Pullup Values Unknown
**Issue:** PULLUP specified but value depends on bus topology.
**Impact:** I2C may be slow or fail.
**Fix:** Verify with scope (should be ~2.2K-4.7KΩ).

---

## LOW RISK ITEMS ℹ️

### 11. Flash Storage Page 14
**Issue:** Last 2 sectors used for params - standard practice.
**Status:** OK.

### 12. DMA_NOSHARE SPI1/SPI4
**Issue:** Correct DMA isolation configured.
**Status:** Good.

### 13. Bootloader Size
**Issue:** 384KB is standard for H7.
**Status:** OK.

### 14. SPI Clock Speeds
**Issue:** ICM42688P: 16MHz (fast), MPU6500: 8MHz (safe).
**Status:** OK.

### 15. Serial Flash Optional
**Issue:** No SPI flash (w25nxx) configured - using SD only.
**Status:** OK for SD card builds.

---

## DESIGN ISSUES TO EXPECT

When powering on for FIRST time, expect:

1. ✅ USB should enumerate
2. ⚠️ RC may fail (inversion issue)
3. ✅ IMUs should detect if SPI pins correct
4. ✅ Barometer should find (try alt address)
5. ✅ Compass should find
6. ✅ SD card should mount (FAT32 required)
7. ✅ PWM should generate
8. ⚠️ High IMU noise if no isolation

---

## PRE-BOARD FIXES NEEDED

Before first power-on, MUST fix:

```
1. Register AP_HW_DIFC with ArduPilot
2. Verify SPI4 pins PE12-14 are correct
3. Fix ADC pin mapping format
4. Add SBUS inversion handling  
```

---

## RECOMMENDED PARAMETERS FOR FIRST BOOT

```bash
# Essential params to set before first flight
SERIAL2_PROTOCOL = 23    # SBUS
BRD_PWM_COUNT = 10      # Use 10 PWMs for quad
CAN_PROTOCOL = 1        # PiccoloCAN
LOG_BITMASK = 6459       # Standard logging
LOG_DISARMED = 1         # Log while disarmed
INS_GYRO_FILTER = 20     # Gyro filter Hz
INS_ACCEL_FILTER = 20     # Accel filter Hz
```

---

## EXPECTED FIRST-BOOT LOG

```
APM: ArduCopter V4.x.x
APM: DIFC board
APM: RC Protocol: RCIN_SBUS
Free RAM: 484KB
Flash: 2048KB
Invensense: ICM42688P found
Invensense: MPU6500 found
BARO: SPL06 found
COMPASS: QMC5883L found
CAN: 1: started
SDCard: mounted
```

---

## IF NOTHING SHOWS

Debug sequence:
```bash
# 1. Check USB
lsusb | grep 0483

# 2. Check DFU  
dfu-util -l

# 3. Try SWD
openocd -f interface/stlink.cfg -f target/stm32h7x.cfg

# 4. Check bootloader flash
stm32flash -S /dev/ttyUSB0

# 5. Try forcing IMU
param set INS_PROBE 1
```

---

This board needs hardware bring-up by experienced developer.
Do not ship to end users without first testing on bench.