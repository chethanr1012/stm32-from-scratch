# Plan: STM32 from scratch on the NUCLEO-F401RE

Self-study project to learn embedded firmware properly on one board, starting
from bare metal with no vendor code and working up to an RTOS-based capstone.

## Context for anyone (or any assistant) picking this up

- **Board:** NUCLEO-F401RE (STM32F401RET6, Cortex-M4F, 512 KB flash, 96 KB RAM).
- **Host:** Windows with Ubuntu 22.04 in WSL 2, VS Code with the WSL extension.
- **Toolchain:** `arm-none-eabi-gcc`, `gdb-multiarch`, `openocd`, `make`.
  No STM32CubeIDE and no HAL until stage 3.
- **Board connection:** the ST-LINK is passed into WSL with usbipd-win.
  After every replug or WSL restart, run in Windows PowerShell:
  `usbipd attach --wsl --busid <BUSID>` (find the BUSID with `usbipd list`).
- **Debug check:** `openocd -f board/st_nucleo_f4.cfg` (GDB server on port 3333).
- **Reference documents:** RM0368 (reference manual), STM32F401xD/xE datasheet,
  UM1724 (Nucleo-64 board manual), PM0214 (Cortex-M4 programming manual).
- **Board facts with page references:** see `docs/notes.md`.

## How I want to work

- I write the code myself. Explain and review rather than generating whole files,
  unless I ask for them.
- Every register, bit, and address should be traced to a document and page.
- One commit per working step, with a short README or notes update each time.

## Status

- [x] Toolchain installed in WSL
- [x] ST-LINK passed through to WSL, OpenOCD connects to the target
- [x] Repo created, `docs/notes.md` with the seven board facts
- [ ] **Next: linker script** (flash at 0x0800 0000, 512 KB; RAM at 0x2000 0000,
      96 KB; initial stack pointer 0x2001 8000)

## Stage 1: Bare metal, no HAL

Folder: `01-bare-metal-blinky/`

- [ ] Linker script: MEMORY regions, `.isr_vector`, `.text`, `.rodata`, `.data`, `.bss`
- [ ] Startup code in C: vector table, reset handler (copy `.data`, zero `.bss`, call `main`)
- [ ] Makefile: build `.elf` and `.bin`, `flash` and `debug` targets using OpenOCD
- [ ] Blinky with registers only: `RCC_AHB1ENR` bit 0, `GPIOA_MODER` bits 11:10 = 01,
      toggle PA5 via `GPIOA_BSRR`, crude delay loop
- [ ] Flash and step through with GDB, inspect registers in memory
- [ ] Replace the delay loop with SysTick (PM0214)
- [ ] Read the user button (PC13, active low) with an EXTI interrupt
- [ ] Clock configuration: HSI 16 MHz to 84 MHz with the PLL

## Stage 2: Own peripheral drivers

Folder: `02-drivers/`

- [ ] GPIO driver (generalise what stage 1 did)
- [ ] UART on USART2 (PA2 TX, PA3 RX, shows up as `/dev/ttyACM0`): polling,
      then interrupt-driven with a ring buffer, then `printf` retargeting
- [ ] Timers: time base, PWM output, input capture
- [ ] ADC: single conversion, then continuous
- [ ] I2C driver with a real sensor (BME280 or MPU6050)
- [ ] SPI driver, verified in loopback on a logic analyzer
- [ ] DMA: UART RX and ADC
- [ ] CAN via external MCP2515 over SPI (the F401 has no CAN peripheral)

Hardware to buy: 8-channel 24 MHz logic analyzer, I2C sensor, breadboard,
jumper wires, MCP2515 module.

## Stage 3: STM32 HAL and CubeMX

Folder: `03-hal/`

- [ ] Redo UART and I2C with HAL
- [ ] Notes comparing the generated code with my own drivers

## Stage 4: FreeRTOS

Folder: `04-freertos/`

- [ ] Port FreeRTOS to the board, two blinking tasks
- [ ] Queues, binary and counting semaphores, mutexes
- [ ] Priority inversion demo and fix
- [ ] ISR-safe APIs, deferred interrupt handling
- [ ] Stack usage and overflow checks

## Stage 5: Professional layer

Folder: `05-professional/`

- [ ] Bootloader with firmware update over UART
- [ ] Low-power modes (sleep, stop, standby) with current measurement on JP6
- [ ] Watchdog (IWDG)
- [ ] Unit tests on the host with Unity/Ceedling
- [ ] CI on GitHub Actions: build and run host tests on every push

## Stage 6: Capstone

Folder: `06-capstone/`

- [ ] Sensor data logger: I2C sensor, SD card over SPI, UART command-line
      interface, FreeRTOS tasks, low-power sleep between samples
- [ ] README with architecture diagram, logic analyzer captures, and lessons learned

## Learning resources (use alongside, not before)

- Miro Samek, "Modern Embedded Systems Programming" (YouTube): startup code,
  vector table, interrupts, RTOS internals
- Low Byte Productions, "Bare Metal Programming Series" (YouTube): open-source
  toolchain, bootloader
- FastBit Embedded Brain Academy (Udemy): driver development, for stage 2.
  Uses the F407 Discovery, so check pins and peripherals against RM0368
- *Making Embedded Systems*, Elecia White
- *Mastering STM32*, Carmine Noviello (for stage 3)

## Repo layout

```
stm32-from-scratch/
├── PLAN.md
├── README.md
├── docs/
│   └── notes.md
├── 01-bare-metal-blinky/
├── 02-drivers/
├── 03-hal/
├── 04-freertos/
├── 05-professional/
└── 06-capstone/
```
