# Board facts: NUCLEO-F401RE (STM32F401RET6)

Facts needed for the bare-metal blinky, each with its source.

Documents:
- RM0368: STM32F401 reference manual (Rev 6)
- DS: STM32F401xD/xE datasheet
- UM1724: STM32 Nucleo-64 boards user manual

## 1. User LED (LD2)

- Connected to **PA5** (Arduino D13).
- **Active high**: I/O high = LED on, I/O low = LED off.
- Source: UM1724 p. 24 (LEDs), p. 45 (Table 16).

## 2. Bus of GPIOA

- **AHB1**.
- Source: RM0368 p. 38 (Table 1, register boundary addresses).

## 3. Base addresses

| Peripheral | Base address | Range |
|---|---|---|
| RCC | 0x4002 3800 | 0x4002 3800 - 0x4002 3BFF |
| GPIOA | 0x4002 0000 | 0x4002 0000 - 0x4002 03FF |

- Source: RM0368 p. 38 (Table 1).

## 4. Enabling the GPIOA clock

- Register: **RCC_AHB1ENR**, offset 0x30, full address 0x4002 3830.
- Bit: **bit 0 (GPIOAEN)**, 1 = clock enabled.
- Source: RM0368 pp. 118-119 (section 6.3.9).

## 5. Configuring PA5 as output

- Register: **GPIOA_MODER**, offset 0x00, full address 0x4002 0000.
- Field: **MODER5[1:0] = bits 11:10** (pin y uses bits 2y+1:2y).
- Value: **01** = general purpose output mode.
  (00 = input, 10 = alternate function, 11 = analog)
- Reset value of GPIOA_MODER is 0xA800 0000 (PA13-PA15 are debug pins),
  so only bits 11:10 may be changed.
- Source: RM0368 p. 158 (section 8.4.1).

## 6. Setting and clearing PA5

- Preferred: **GPIOA_BSRR**, offset 0x18 (write-only, atomic).
  - Bit 5 (BS5): write 1 to set PA5.
  - Bit 21 (BR5): write 1 to clear PA5.
- Alternative: **GPIOA_ODR**, offset 0x14, bit 5 (needs read-modify-write).
- Source: RM0368 pp. 160-161 (sections 8.4.6, 8.4.7).

## 7. Memory layout

| Region | Start | End | Size |
|---|---|---|---|
| Flash | 0x0800 0000 | 0x0807 FFFF | 512 KB |
| SRAM | 0x2000 0000 | 0x2001 7FFF | 96 KB |

- Initial stack pointer (end of SRAM + 1): 0x2001 8000.
- Note: 0xE000 0000 is the Cortex-M4 internal peripherals, not flash.
- Source: DS p. 51 (Figure 14, memory map), p. 1 (sizes).