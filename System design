# 基于昇腾310B手势识别控制WS2812B彩灯带

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-OrangePi%20AIpro%20%7C%20STM32-brightgreen)]()

---

## 📋 项目计划书（06-30提交）

### 项目概述
本项目利用OrangePi AIpro（昇腾310B）运行YOLOv10手势识别模型，识别结果通过串口发送至STM32，驱动WS2812B彩灯带实现颜色及动态特效的切换。项目由两人合作完成。

### 时间节点与任务

| 截止时间 | 阶段任务 | 交付物 |
|----------|----------|--------|
| 06-30 09:00 | 环境搭建、物料准备 | 项目计划书 |
| 07-02 09:00 | 模型转换（ONNX→OM）、NPU推理 | 设计文档 |
| 07-06 15:00 | STM32灯带驱动、串口通信调通 | 中期检查报告 |
| 07-09 09:00 | 全流程闭环联调 | 第一次验收演示 |
| 07-10 09:00 | 最终优化与录像 | 第二次验收 |
| 验收后一周内 | 个人总结 | 个人报告（LaTeX） |

### 手势-灯光映射

| 手势 | 串口指令 | 灯带动作 |
|------|----------|----------|
| 点赞 | `CMD:ACC` | 暖色高亮 |
| 握拳 | `CMD:DEC` | 冷色低亮 |
| 掌心 | `CMD:STOP` | 全灭 |
| 剪刀手 | `CMD:REV` | 彩虹跑马灯 |

### 硬件采购清单

| 名称 | 规格 | 总价 |
|------|------|------|
| AI主板 | OrangePi AIpro (昇腾310B) | ¥680 |
| MCU | STM32F103C8T6 核心板 | ¥25 |
| 摄像头 | USB 720P (支持MJPG) | ¥80 |
| 灯带 | WS2812B (5V, 30颗/米) | ¥18 |
| 主板电源 | 12V 3A适配器 | ¥35 |
| 灯带电源 | 5V 3A独立适配器 | ¥25 |
| 串口模块 | CH340 USB转TTL | ¥12 |
| 电容 | 1000µF/16V | ¥2 |
| **总计** | | **¥927** |

> ⚠️ 灯带必须独立供电，严禁从主板取电。

### 两人分工

- **成员A（算法与上位机）**：模型转换、NPU推理脚本、串口发送逻辑、Flask推流。
- **成员B（嵌入式与硬件）**：STM32驱动（SPI+DMA）、UART中断、硬件接线、灯带调试。

### 降级方案

- WebRTC推流 → Flask+MJPEG
- FreeRTOS → 裸机中断+轮询
- 模型转换报错 → 参考华为官方YOLO样例

### 准备状态自查

- [ ] 硬件已下单，预计7月1日前到货
- [ ] CANN 7.0 环境已安装
- [ ] VS Code + ARM-GCC 已测试

---

## 📐 系统设计文档（07-02提交）

### 1. 物料到位情况

| 物料 | 状态 | 备注 |
|------|------|------|
| OrangePi AIpro 主板 | ✅ 已到货 | 昇腾310B，带散热 |
| STM32F103C8T6 核心板 | ✅ 已到货 | 蓝丸板，排针已焊 |
| USB摄像头（MJPG 720P） | ✅ 已到货 | 支持MJPG格式 |
| WS2812B 灯带（5V，30颗/米） | ✅ 已到货 | 裸线头版本 |
| 12V 3A 电源适配器 | ✅ 已到货 | 主板供电 |
| 5V 3A 电源适配器 | ✅ 已到货 | 灯带独立供电 |
| CH340 USB转TTL | ✅ 已到货 | 串口调试 |
| 1000µF/16V 电容 | ✅ 已到货 | 2个 |
| 杜邦线（公/母） | ✅ 已到货 | 各40根 |

### 2. 系统总体架构

```mermaid
flowchart LR
    A[USB摄像头] --> B[昇腾310B NPU推理]
    B --> C[串口 UART]
    C --> D[STM32F103C8T6]
    D --> E[SPI+DMA]
    E --> F[WS2812B灯带]
    B --> G[Flask MJPEG推流]
    G --> H[浏览器实时预览]
flowchart TD
    subgraph OrangePi["OrangePi AIpro (昇腾310B)"]
        TX[UART_TX Pin8]
        RX[UART_RX Pin10]
        GND1[GND]
    end

    subgraph STM32["STM32F103C8T6"]
        PA10[PA10 RX]
        PA9[PA9 TX]
        GND2[GND]
        PA7[PA7 SPI1_MOSI]
    end

    subgraph LED["WS2812B灯带"]
        DIN[DIN]
        VCC[VCC 5V]
        GND3[GND]
    end

    TX --> PA10
    PA9 --> RX
    GND1 --- GND2
    GND2 --- GND3

    PA7 -->|串联300~500Ω| DIN
    VCC -->|独立5V 3A电源| PWR[5V电源]
    GND3 --> PWR

    CAP[1000µF/16V电容] -- 并联 --> VCC
    CAP -- 并联 --> GND3
flowchart TD
    subgraph 昇腾端["昇腾端 (Python)"]
        A[Camera采集] --> B[预处理 Letterbox]
        B --> C[NPU推理 YOLOv10 OM]
        C --> D[后处理 解析+坐标映射]
        D --> E[画框 显示]
        E --> F[串口发送 PySerial]
        E --> G[MJPEG推流 Flask]
    end

    subgraph STM32端["STM32端 (C)"]
        H[UART中断接收] --> I[指令解析]
        I --> J[灯带驱动 SPI+DMA]
        J --> K[WS2812B显示]
    end

    F -->|UART| H
flowchart LR
    subgraph 昇腾主循环["昇腾主循环 (单线程)"]
        S1[采集] --> S2[推理] --> S3[后处理] --> S4[画框推流] --> S5[串口发送]
        S5 -.->|去重| S5
    end

    subgraph STM32["STM32 (裸机中断+轮询)"]
        T1[UART接收中断] --> T2[环形缓冲]
        T2 --> T3[主循环解析]
        T3 --> T4[执行灯带显示]
    end

    S5 -->|串口指令| T1
flowchart LR
    root[GestureControl_LED] --> README[README.md]
    root --> LICENSE[LICENSE]
    root --> requirements[requirements.txt]
    root --> ascend[ascend/]
    root --> stm32[stm32/]
    root --> web[web/]

    ascend --> infer[infer.py]
    ascend --> utils[model_utils.py]
    ascend --> config[config.py]

    stm32 --> Core[Core/ HAL库]
    stm32 --> Inc[Inc/]
    stm32 --> Src[Src/]
    stm32 --> Makefile[Makefile]

    Inc --> ws2812_h[ws2812.h]
    Inc --> uart_h[uart_handler.h]

    Src --> main_c[main.c]
    Src --> ws2812_c[ws2812.c]
    Src --> uart_c[uart_handler.c]

    web --> flask[flask_stream.py]
---

## 🚀 部署步骤（双人协作版）

### 成员A（昇腾端）
1. 安装依赖：`pip install -r requirements.txt`
2. 将YOLOv10导出ONNX，使用ATC转为OM（适配Ascend310B）
3. 修改`ascend/config.py`中的`SERIAL_PORT`（如`/dev/ttyS0`）
4. 运行主程序：`python ascend/infer.py`
5. （可选）运行推流：`python web/flask_stream.py`

### 成员B（STM32端）
1. 使用STM32CubeMX生成Makefile工程（USART1中断，SPI1+DMA）
2. 将`stm32/Inc/`和`stm32/Src/`文件加入工程
3. 在`main.c`中调用`UART_Init_IT()`和`WS2812_Turn_Off()`
4. 编译：`make`，烧录（ST-Link或串口ISP）
5. 串口助手发送`CMD:ACC\r\n`测试灯带

---

## 📁 项目文件结构
---

## ❓ 常见问题

- **摄像头无法打开**：检查`/dev/video*`，确认支持MJPG。
- **NPU推理失败**：确认OM模型芯片类型，用`msame`工具测试。
- **灯带不亮**：检查5V供电及SPI波形（示波器测PA7），确保DMA配置正确。
- **串口无响应**：核对设备名和波特率，用USB转TTL分别测试两端。
- **推流卡顿**：降低分辨率，关闭OpenCV显示窗口。

---

## 📄 许可与致谢

- MIT License
- 华为昇腾社区、YOLOv10官方仓库、开源SPI+DMA方案

---

**文档版本**：v1.0 | **最后更新**：2026-07-01 | **维护者**：成员A & 成员B
