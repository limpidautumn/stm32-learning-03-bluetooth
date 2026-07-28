# stm32-learning-03-bluetooth

STM32 练习代码 03：使用蓝牙控制 LED。

[English](./README.md) | [中文](./README.zh-CN.md)

一个蓝牙串口 LED 控制器，运行在 [Keysking STM32 开发板](https://docs.keysking.com/docs/stm32/resourcePack/) 上。它通过蓝牙从主机接收自定义二进制命令，以控制 RGB LED 的亮灭，并带有校验和验证的请求/响应数据包。

## 概述

本项目在 STM32F103C8T6 开发板上实现了一个蓝牙串口透传（UART）远程控制 RGB LED 的功能。它使用自定义通信协议，通过 USART3 配合 DMA 进行高效的数据传输。系统从蓝牙模块（BT24）接收控制数据包，更新相应 LED 的状态，然后发送一个响应数据包以确认新状态。

本项目是基于 [Keysking STM32 教程](https://space.bilibili.com/6100925/lists/1025423) 的学习实践，设计运行于配备 BT24 蓝牙透传模块的开发板上。

## 实现细节

- **MCU**：STM32F103C8T6
- **外设**：
  - USART3（TX：PB10，RX：PB11），配置为 9600 波特率，8N1。
  - DMA1（通道 2 用于 TX，通道 3 用于 RX），普通模式。
  - GPIO 输出控制 RGB LED：红（PB0）、绿（PA7）、蓝（PA6）。
- **协议**（自定义）：
  - 具体规格参见文件 [protocol.zh-CN.md](./docs/protocol.zh-CN.md)。
  - **请求数据包**（蓝牙主机 → MCU）：
    - 起始字节 `0x0C`、总长度、载荷（每个 LED 的索引+状态）、结束字节 `0x0A`、校验和。
    - LED 索引：0=红，1=绿，2=蓝；状态：`0xFF`=开，`0x00`=关。
  - **响应数据包**（MCU → 主机）：
    - 起始字节 `0x1C`、总长度、载荷（当前 LED 状态）、结束字节 `0x1A`、校验和。
- **流程**：
  - 使用 `HAL_UARTEx_ReceiveToIdle_DMA` 进行非阻塞接收。
  - 在 `RxEventCallback` 中验证收到的数据包（长度、校验和、起始/结束字节）。
  - 若有效，解析载荷并更新 `led_state[]`，然后通过 DMA 发送响应。
  - 主循环中的 `updateLed()` 函数将状态写入 GPIO 引脚。

## 项目结构

```text
stm32-learning-03-bluetooth/
├── CMakeLists.txt
├── Core
│   └── Src
│       └── main.c     # 主程序
├── LICENSE
├── README.md
├── docs
│   ├── guide.md       # 开发笔记（硬件连接、协议等）
│   └── protocol.md    # 通信协议定义
└── stm32-learning-03-bluetooth.ioc  # STM32CubeMX 项目文件
```

## 学习目标
- 配置带 DMA 的 UART 通信，实现高效数据交换。
- 设计并实现一个带校验和验证的简单二进制协议。
- 处理 DMA 中断和 HAL 回调，实现异步接收。
- 根据接收数据控制 GPIO 输出。
- 理解蓝牙串口模块（BT24）与 STM32 的集成。
