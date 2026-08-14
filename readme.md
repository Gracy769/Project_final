# Low-Cost AI-Powered Optical Inspection System for PCB Assembly

> **Problem Statement:** PS82 — Low-cost Optical Inspection for PCB Assembly
> **Category:** Manufacturing & Quality Control
> **Domain:** Edge AI / Embedded Systems / Computer Vision

---

## Table of Contents

- [1. Problem Statement](#1-problem-statement)
- [2. Existing Solutions and Their Limitations](#2-existing-solutions-and-their-limitations)
- [3. Proposed Solution](#3-proposed-solution)
- [4. System Architecture](#4-system-architecture)
- [5. Circuit Connection Diagram](#5-circuit-connection-diagram)
- [6. Component List](#6-component-list)
- [7. AI Model Pipeline](#7-ai-model-pipeline)
- [8. Comparison with Existing Approaches](#8-comparison-with-existing-approaches)
- [9. Expected Outcomes](#9-expected-outcomes)
- [10. Project Timeline](#10-project-timeline)
- [11. Estimated Budget](#11-estimated-budget)
- [12. References](#12-references)

---

## 1. Problem Statement

In India's MSME electronics manufacturing sector, Printed Circuit Board (PCB) quality inspection remains a predominantly manual process. Trained operators visually examine solder joints and component placement using magnifying glasses, typically achieving a throughput of 150–200 boards per hour. This manual approach introduces the following challenges:

- **High defect escape rate** — Operator fatigue results in a 10–30% miss rate for solder defects.
- **Inconsistent quality** — Inspection accuracy varies across operators and shifts.
- **Throughput bottleneck** — Manual inspection is the slowest stage in the PCB assembly line.
- **Skilled labour dependency** — Trained inspection operators are difficult to hire and retain.

These issues directly impact product reliability, customer returns, and manufacturing costs for small-scale electronics manufacturers.

---

## 2. Existing Solutions and Their Limitations

| Solution | Description | Limitations |
|---|---|---|
| **Manual Visual Inspection** | Operators use magnifying lenses to examine solder joints | Subjective, slow (150–200 boards/hr), 10–30% defect miss rate |
| **Commercial AOI Machines** (Koh Young, Mirtec, Omron) | Industrial automated optical inspection systems with structured lighting and multi-angle cameras | Cost: ₹15–50 lakhs per unit; requires dedicated floor space; not accessible to MSMEs |
| **Desktop AOI Software** (VisionBot, Macula) | PC-based software that analyses images from external cameras | Requires a dedicated PC, high-resolution industrial camera, and internet/cloud connectivity |
| **Cloud-based AI Inspection** | Upload PCB images to cloud for AI inference | Requires stable internet connectivity; data privacy concerns; recurring subscription costs |

> **Gap Identified:** There is no affordable, standalone, offline solution that small-scale electronics manufacturers can deploy without requiring a PC, internet, or expensive industrial equipment.

---

## 3. Proposed Solution

An embedded AI-powered PCB inspection system built on the **Sipeed Maix (Kendryte K210)** microcontroller platform. The system captures high-resolution images of PCB assemblies using a 5MP autofocus camera, performs real-time defect detection using an on-device neural network, and displays results on an integrated LCD.

### 3.1 Key Features

| Feature | Description |
|---|---|
| 🔌 **Fully Offline** | All processing occurs on the microcontroller. No internet or cloud dependency. |
| ⚡ **Real-Time Detection** | Inference speed of 30–60ms per frame using the K210's hardware Neural Network Processor (KPU). |
| 🎯 **Multi-Class Detection** | Identifies solder bridges, missing components, misaligned components, and cold solder joints. |
| 🖥️ **Visual Output** | Displays live camera feed with bounding boxes around detected defects on 2.4" colour LCD. |
| 💰 **Low Cost** | Total system cost under ₹3,000, compared to ₹15–50 lakhs for commercial alternatives. |
| 🎒 **Portable** | Compact form factor, battery-operable, suitable for deployment on any workbench. |

### 3.2 Defect Categories

| Defect Type | Description | Severity |
|---|---|:---:|
| Solder Bridge | Unintended solder connection between adjacent pads | 🔴 Critical |
| Missing Component | Empty footprint where a component should be placed | 🔴 Critical |
| Component Misalignment / Tombstone | Component shifted or lifted from its designated position | 🟠 Major |
| Cold Solder Joint | Dull, irregular solder connection indicating poor wetting | 🟠 Major |

---

## 4. System Architecture

```
┌────────────────────────────────────────────────────────────────────┐
│                       SYSTEM ARCHITECTURE                          │
│                                                                      │
│  ┌───────────────┐                                                  │
│  │  OV5640       │    DVP Interface    ┌────────────────────────┐   │
│  │  Camera Module│───────────────────▶│  Kendryte K210 SoC      │   │
│  │  (5MP, AF)    │                    │                         │   │
│  └───────────────┘                    │  ┌──────────────────┐   │   │
│                                        │  │ RISC-V Dual Core │   │   │
│  ┌───────────────┐                    │  │ @ 400 MHz        │   │   │
│  │  LED Ring     │    GPIO            │  └──────────────────┘   │   │
│  │  Light Module │◀──────────────────│                          │   │
│  │  (WS2812B)    │                    │  ┌──────────────────┐   │   │
│  └───────────────┘                    │  │ KPU (Neural      │   │   │
│                                        │  │ Network          │   │   │
│  ┌───────────────┐                    │  │ Processor)       │   │   │
│  │  Push Button  │    GPIO            │  │ 0.8 TOPS         │   │   │
│  │  (Trigger)    │───────────────────▶│  └──────────────────┘   │   │
│  └───────────────┘                    │                         │   │
│                                        │  ┌──────────────────┐   │   │
│  ┌───────────────┐    SPI Interface   │  │ 8 MB SRAM        │   │   │
│  │  2.4" TFT LCD │◀──────────────────│  └──────────────────┘   │   │
│  │  (ST7789)     │                    │                         │   │
│  └───────────────┘                    └────────────────────────┘   │
│                                               │                     │
│  ┌───────────────┐    GPIO                    │                     │
│  │  Piezo Buzzer │◀───────────────────────────┘                     │
│  └───────────────┘                                                  │
│                                                                      │
│  ┌───────────────┐                                                  │
│  │  SD Card      │    SPI Interface                                 │
│  │  Module       │◀──── Inspection log storage                      │
│  └───────────────┘                                                  │
└────────────────────────────────────────────────────────────────────┘
```

### 4.1 Inspection Workflow

```
┌──────────┐     ┌──────────────┐     ┌───────────────┐     ┌──────────────┐
│  Place   │     │  Press       │     │  LED Ring     │     │  OV5640      │
│  PCB on  │────▶│  Trigger     │────▶│  Illuminates  │────▶│  Captures    │
│  Jig     │     │  Button      │     │  PCB Surface  │     │  Image       │
└──────────┘     └──────────────┘     └───────────────┘     └──────┬───────┘
                                                                     │
                                                                     ▼
┌──────────┐     ┌──────────────┐     ┌───────────────┐     ┌──────────────┐
│  Log to  │     │  Display on  │     │  Classify     │     │  K210 KPU    │
│  SD Card │◀────│  LCD with    │◀────│  Defect Type  │◀────│  Runs YOLO   │
│          │     │  Bounding Box│     │  & Location   │     │  Inference   │
└──────────┘     └──────────────┘     └───────────────┘     └──────────────┘
                                                              (30-60 ms)
```

---

## 5. Circuit Connection Diagram

```
                         Sipeed Maix Bit (K210)
                        ┌──────────────────────┐
                        │                       │
   OV5640 Camera ──────▶│ DVP Port (24-pin FPC) │
   (via FPC cable)      │                       │
                        │                       │
   2.4" TFT LCD ◀──────│ SPI Port              │
   (via FPC cable)      │  LCD_CS   → IO36      │
                        │  LCD_RST  → IO37      │
                        │  LCD_DC   → IO38      │
                        │  LCD_WR   → IO39      │
                        │                       │
   LED Ring (WS2812B)   │                       │
   ┌────────────┐       │                       │
   │ VCC ───────┼───────│ 5V                    │
   │ GND ───────┼───────│ GND                   │
   │ DIN ───────┼───────│ IO24 (GPIO)           │
   └────────────┘       │                       │
                        │                       │
   Push Button          │                       │
   ┌────────────┐       │                       │
   │ Terminal 1 ┼───────│ IO16 (GPIO, Pull-Up)  │
   │ Terminal 2 ┼───────│ GND                   │
   └────────────┘       │                       │
                        │                       │
   Piezo Buzzer         │                       │
   ┌────────────┐       │                       │
   │ + Terminal ┼───────│ IO25 (GPIO)           │
   │ - Terminal ┼───────│ GND                   │
   └────────────┘       │                       │
                        │                       │
   SD Card Module       │                       │
   ┌────────────┐       │                       │
   │ CS   ──────┼───────│ IO29                  │
   │ MOSI ──────┼───────│ IO30                  │
   │ MISO ──────┼───────│ IO31                  │
   │ SCK  ──────┼───────│ IO32                  │
   │ VCC  ──────┼───────│ 3.3V                  │
   │ GND  ──────┼───────│ GND                   │
   └────────────┘       │                       │
                        │                       │
   Power Supply         │                       │
   ┌────────────┐       │                       │
   │ USB-C      ┼───────│ USB-C Port (5V/500mA) │
   │ or 18650   │       │                       │
   │ + TP4056   │       │                       │
   └────────────┘       └───────────────────────┘
```

---

## 6. Component List

| S.No. | Component | Specification | Qty | Est. Cost (₹) |
|---|---|---|:---:|---:|
| 1 | Sipeed Maix Bit Development Board | Kendryte K210, Dual-core RISC-V @ 400 MHz, 8 MB SRAM, KPU (0.8 TOPS), USB-C | 1 | 1,400 |
| 2 | OV5640 Camera Module | 5MP, Autofocus, DVP Interface, 24-pin FPC | 1 | 400 |
| 3 | 2.4" TFT LCD Display | ST7789 Driver, 320×240, SPI | 1 | 250 |
| 4 | WS2812B LED Ring Light | 12 Addressable RGB LEDs, 5V | 1 | 150 |
| 5 | Micro SD Card | 8 GB, Class 10 | 1 | 150 |
| 6 | SD Card Module | SPI Interface, 3.3V/5V Compatible | 1 | 80 |
| 7 | Active Piezo Buzzer | 5V, Through-hole | 1 | 20 |
| 8 | Tactile Push Button | Momentary, 6×6 mm | 1 | 10 |
| 9 | Breadboard | 830 Tie Points | 1 | 100 |
| 10 | Jumper Wires | Male-to-Female, 20 cm, Assorted | 20 | 60 |
| 11 | 18650 Li-ion Battery | 3.7V, 2600 mAh | 1 | 150 |
| 12 | TP4056 Charging Module | Micro-USB, 1A, DW01 Protection | 1 | 30 |
| 13 | Enclosure | 3D Printed or Laser-Cut Acrylic | 1 | 150 |
| | | | **Total** | **₹2,950** |

---

## 7. AI Model Pipeline

### 7.1 Dataset

| Source | Description | Size |
|---|---|---|
| PKU PCB Defect Dataset | 1,386 annotated images with 6 defect categories | Open Source |
| DeepPCB Dataset | 1,500 template-defective image pairs | Open Source |
| Custom Dataset | Images captured using the OV5640 camera under actual operating conditions | 200–500 images |
| Data Augmentation | Rotation, flipping, brightness adjustment, cropping | 5x–10x expansion |

### 7.2 Model Architecture

| Parameter | Value |
|---|---|
| Model | TinyYOLOv2 (adapted for K210 KPU) |
| Input Resolution | 224 × 224 × 3 (RGB) |
| Output | Bounding box coordinates + class label + confidence score |
| Quantisation | INT8 (required by K210 KPU) |
| Model Size | 200–500 KB |
| Inference Speed | 30–60 ms per frame on KPU |

### 7.3 Training and Deployment Pipeline

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Dataset      │     │ Train on     │     │ Convert to   │     │ Deploy to    │
│ Collection   │────▶│ Google Colab │────▶│ K210 KModel  │────▶│ Sipeed Maix  │
│ & Annotation │     │ (Free GPU)   │     │ (.kmodel)    │     │ via SD Card  │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
                      • TinyYOLOv2        • NNCase Compiler     • MaixPy loads
                      • Transfer Learning • INT8 Quantisation     model from SD
                      • 50–100 epochs     • K210 KPU optimised  • Real-time
                                                                   inference
```

### 7.4 Software Stack

| Layer | Technology |
|---|---|
| Firmware | MaixPy (MicroPython for K210) |
| AI Framework | TinyYOLOv2 on K210 KPU |
| Model Compiler | NNCase (Kendryte model converter) |
| Training Environment | Google Colab (free tier) / Local with GPU |
| Annotation Tool | LabelImg / Roboflow |
| Camera Interface | MaixPy built-in camera driver |

---

## 8. Comparison with Existing Approaches

| Parameter | Manual Inspection | Commercial AOI | Proposed System |
|---|---|---|---|
| **Cost** | ₹0 (operator salary ongoing) | ₹15–50 Lakhs | **₹2,950 (one-time)** |
| **Throughput** | 150–200 boards/hr | 1,000+ boards/hr | 500–800 boards/hr |
| **Accuracy** | 70–90% (operator dependent) | 95–99% | 85–93% (estimated) |
| **Internet Required** | No | Some models require cloud | **No** |
| **Portability** | Yes | No (fixed installation) | **Yes** |
| **Setup Time** | None | Days (calibration) | **Minutes** |
| **MSME Accessible** | Yes | No (cost prohibitive) | **Yes** |

---

## 9. Expected Outcomes

1. A functional prototype capable of detecting 4 categories of PCB assembly defects in real-time.
2. Inference latency under 60 ms per frame, enabling near real-time inspection.
3. Detection accuracy of 85–93% on the target defect categories.
4. Total hardware cost under ₹3,000, making it accessible to MSME electronics manufacturers.
5. Fully offline operation with no dependency on internet connectivity or external computing resources.

---

## 10. Project Timeline

| Week | Activity |
|---|---|
| **Week 1** | Component procurement. Development environment setup. Camera and display integration testing. |
| **Week 2** | Dataset collection and annotation. Reference PCB image capture using OV5640. Data augmentation. |
| **Week 3** | Model training on Google Colab. K210 model conversion using NNCase. Deployment and on-device testing. |
| **Week 4** | Full system integration. Enclosure fabrication. Accuracy evaluation. Presentation preparation. |

---

## 11. Estimated Budget

| Category | Amount (₹) |
|---|---:|
| Electronic Components | 2,650 |
| Enclosure and Mechanical Parts | 150 |
| Miscellaneous (wires, connectors, PCB samples for testing) | 150 |
| **Total** | **₹2,950** |

---

## 12. References

1. [PKU PCB Defect Dataset](https://github.com/Ixiaohuihuihui/Tiny-Defect-Detection-for-PCB)
2. [DeepPCB Dataset](https://github.com/tangsanli5201/DeepPCB)
3. [Kendryte K210 Technical Reference Manual](https://kendryte.com)
4. [MaixPy Documentation](https://wiki.sipeed.com/soft/maixpy/en/)
5. [NNCase Model Compiler](https://github.com/kendryte/nncase)
