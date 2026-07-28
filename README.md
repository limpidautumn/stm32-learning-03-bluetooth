# stm32-learning-03-bluetooth

STM32 practice code #03: Bluetooth-controlled LED.

[English](./README.md) | [中文](./README.zh-CN.md)

A Bluetooth serial-to-LED controller running on a [Keysking STM32 development board](https://docs.keysking.com/docs/stm32/resourcePack/). It receives custom binary commands from the host via Bluetooth to toggle the RGB LEDs, with checksum-verified request/response packets.

## Overview

This project implements a Bluetooth serial pass-through (UART) remote control for an RGB LED on an STM32F103C8T6 board. It uses a custom communication protocol over USART3 with DMA for efficient data transfer. The system receives control packets from a Bluetooth module (BT24) and updates the LED states accordingly, then sends back a response packet confirming the new states.

This project is a learning exercise based on [Keysking's STM32 tutorial](https://space.bilibili.com/6100925/lists/1025423). It is designed to run on a development board equipped with a BT24 Bluetooth transparent transmission module.

## Implementation Details

- **MCU**: STM32F103C8T6
- **Peripherals**:
  - USART3 (TX: PB10, RX: PB11) configured at 9600 baud, 8N1.
  - DMA1 (Channel 2 for TX, Channel 3 for RX) in normal mode.
  - GPIO outputs for RGB LEDs: Red (PB0), Green (PA7), Blue (PA6).
- **Protocol** (custom):
  - Refer to the specifications in file [protocol.md](./docs/protocol.md).
  - **Request packet** (from Bluetooth host to MCU):
    - Start byte `0x0C`, total length, payload (index + state) for each LED, end byte `0x0A`, checksum.
    - LED index: 0=Red, 1=Green, 2=Blue; state: `0xFF` = ON, `0x00` = OFF.
  - **Response packet** (MCU to host):
    - Start byte `0x1C`, total length, payload (current LED states), end byte `0x1A`, checksum.
- **Flow**:
  - `HAL_UARTEx_ReceiveToIdle_DMA` is used for non‑blocking reception.
  - On `RxEventCallback`, the received packet is validated (length, checksum, start/end bytes).
  - If valid, the payload is parsed to update `led_state[]`, and a response is transmitted via DMA.
  - The `updateLed()` function in the main loop writes the states to the GPIO pins.

## Project Structure

```text
stm32-learning-03-bluetooth/
├── CMakeLists.txt
├── Core
│   └── Src
│       └── main.c   # main program
├── LICENSE
├── README.md
├── docs
│   ├── guide.md     # dev notes (hardware connections, protocols, etc.)
│   └── protocol.md  # communication protocol definition
└── stm32-learning-03-bluetooth.ioc  # STM32CubeMX project file
```

## Learning Objectives
- Configure UART communication with DMA for efficient data exchange.
- Design and implement a simple binary protocol with checksum validation.
- Handle DMA interrupts and HAL callbacks for asynchronous reception.
- Control GPIO output based on received data.
- Understand the integration of a Bluetooth serial module (BT24) with an STM32.
