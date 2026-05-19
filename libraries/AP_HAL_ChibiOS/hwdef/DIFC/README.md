# DIFC Board Support Files for ArduPilot

Board: DIFC (Custom STM32H743VIT6 Flight Controller)
MCU: STM32H743VIT6 (2MB Flash, 1MB RAM)
Crystal: 8MHz

---

## VALIDATED PIN MAPPINGS (Final)

### USB
- PA11: OTG_FS_DM (OTG1)
- PA12: OTG_FS_DP (OTG1)
- PA8: VBUS sense

### SPI Buses
| Bus | Function | SCK | MISO | MOSI | CS |
|-----|----------|-----|------|------|-----|
| SPI4 | ICM42688P (Primary) | PE12 | PE13 | PE14 | **PD4** (FIXED) |
| SPI1 | MPU6500 (Secondary) | PA5 | PA6 | PA7 | PA4 |
| SPI2 | AT7456E OSD | PD3 | PB14 | **PB15** (FIXED) | PB12 |
| SPI3 | Expansion | PB3 | PB4 | PB5 | - |

### UARTs (5 + USB)
| UART | Purpose | RX | TX | Notes |
|------|---------|-----|-----|---------|
| OTG1 | USB VCP | PA11 | PA12 | Primary |
| USART2 | RCIN/SBUS | PD6 | PD5 | Default RCIN |
| UART4 | GPS | PB8 | PB9 | |
| UART6 | Expansion | PC7 | PC6 | |
| UART7 | Expansion | PE7 | PE8 | |
| UART8 | Expansion | PE0 | PE1 | |

### I2C
| Bus | Function | SCL | SDA |
|-----|----------|-----|-----|
| I2C1 | QMC5883L Compass | PB6 | PB7 |
| I2C2 | SPL06 Barometer | PB10 | PB11 |

### CAN
- PD0: CAN1_RX
- PD1: CAN1_TX

### SDMMC (Native SDIO)
- PC8-PC11: D0-D3
- PC12: CK
- PD2: CMD

### PWM Outputs (12)
| Output | Pin | Timer | Channel | DMA | BDShot |
|--------|-----|-------|---------|-----|--------|
| PWM1 | PB0 | TIM3 | CH3 | Y | Y |
| PWM2 | PB1 | TIM3 | CH4 | Y | Y |
| PWM3 | PA0 | TIM5 | CH1 | Y | N |
| PWM4 | PA1 | TIM5 | CH2 | Y | N |
| PWM5 | PA2 | TIM5 | CH3 | Y | N |
| PWM6 | PA3 | TIM5 | CH4 | Y | N |
| PWM7 | PD12 | TIM4 | CH1 | Y | Y |
| PWM8 | PD13 | TIM4 | CH2 | Y | Y |
| PWM9 | PD14 | TIM4 | CH3 | Y | Y |
| PWM10 | PD15 | TIM4 | CH4 | Y | Y |
| PWM11 | PE5 | TIM15 | CH1 | N | N |
| PWM12 | PE6 | TIM15 | CH2 | N | N |

### Other
- PC13: Buzzer
- PC0: BATT_VOLTAGE_SENS (ADC)
- PC1: BATT_CURRENT_SENS (ADC)

---

## TODO: Unresolved Items

### Critical
1. ADC Scaling - PC0/PC1 placeholders need actual divider ratios
2. WS2812 LED pin - Not yet assigned (verify if used)
3. Board ID - Need unique AP_HW_DIFC from ArduPilot

### IMU EXTI (Cannot resolve without schematic):
- ICM42688P EXTI: "IMU1_SPI4_EXTI2"
- MPU6500 EXTI: "IMU2_SPI1_EXTI1"

---

## Static Validation

### Confirmed Valid:
- No duplicate GPIO pins (after fixing conflicts)
- All UART pins valid for STM32H743
- All SPI AF mappings valid
- All timer/channels valid for H7
- SPI1/SPI4 DMA isolation

### Trade-offs:
- UART1 removed to avoid VBUS conflict
- Using SPI4 on PE12-15 to avoid SDMMC conflicts
- BDShot: PWM1-2 (TIM3), PWM7-10 (TIM4)

---

## Build Commands

./waf configure --board DIFC
./waf copter

---

## Files
libraries/AP_HAL_ChibiOS/hwdef/DIFC/
  hwdef.dat       - Production definition
  hwdef-bl.dat   - Bootloader
  README.md     - This