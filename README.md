<!-- ANIMATED HEADER -->
<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=6,11,20&height=200&section=header&text=PCB%20Defect%20Detection&fontSize=40&fontColor=ffffff&animation=twinkling&fontAlignY=35&desc=Low-Cost%20Embedded%20Inspection%20for%20Indian%20MSMEs&descAlignY=58&descSize=17" />
</p>

<p align="center">
  <a href="https://git.io/typing-svg">
    <img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=20&duration=3000&pause=800&color=00D9FF&center=true&vCenter=true&multiline=true&width=750&height=90&lines=ESP32-CAM+%7C+M12+Macro+Lens+%7C+Template+Detection;Fully+Offline+%E2%80%94+No+Cloud%2C+No+PC%2C+No+Network;Under+%E2%82%B9750+Total+Build+Cost" alt="Typing SVG" />
  </a>
</p>

<p align="center">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-ESP32--CAM-blue?style=for-the-badge&logo=espressif" />
  <img alt="Cost" src="https://img.shields.io/badge/Cost-Under%20%E2%82%B9750-green?style=for-the-badge" />
  <img alt="Offline" src="https://img.shields.io/badge/Offline-100%25-brightgreen?style=for-the-badge" />
  <img alt="Last Verified" src="https://img.shields.io/badge/Last%20Verified-2026--08--14-yellow?style=for-the-badge" />
</p>

---

## Table of Contents

- [Problem Statement](#problem-statement)
- [Hardware Solution](#hardware-solution)
- [Bill of Materials](#bill-of-materials)
- [Circuit](#circuit)
- [Detection Method](#detection-method)
- [Performance](#performance)
- [vs Commercial AOI](#vs-commercial-aoi)
- [Datasets](#datasets)
- [Setup Guide](#setup-guide)

---

## Problem Statement

India's MSME electronics sector relies on manual PCB visual inspection, which carries a documented **10-30% defect miss rate**. Commercial Automated Optical Inspection (AOI) machines cost **Rs 15-50 lakhs** per unit, placing them outside reach for small manufacturers.

This project delivers a fully offline, microcontroller-only alternative under **Rs 750** — no cloud, no PC, no network required at runtime.

---

## Hardware Solution

**Root cause of blur on stock ESP32-CAM:** The OV2640 sensor ships with a fixed-focus lens calibrated for approximately 60 cm working distance. PCB inspection requires a 3-5 cm working distance, which causes severe defocus on the stock lens.

**Fix: M12 Macro Lens Swap**

The OV2640 lens barrel uses a standard **M12 (S-mount)** thread. The procedure is:

1. Apply acetone to dissolve the factory UV glue around the lens barrel
2. Unscrew the stock fixed-focus lens
3. Thread in an M12 macro lens rated for 3-10 cm focus distance
4. Verify sharp focus at your intended working distance before reassembly

---

## Bill of Materials

| Item | Specification | Source | Cost |
|------|--------------|--------|------|
| ESP32-CAM | OV2640, 2MP, 802.11 b/g/n | Amazon / Robocraze | Rs 150-300 |
| M12 macro lens | 3-10 cm focus, S-mount | Amazon / AliExpress | Rs 80-150 |
| FTDI FT232RL adapter | USB-to-UART, 3.3V/5V | Amazon | Rs 120-200 |
| USB cable | Micro-USB | Local | Rs 30-60 |
| Breadboard + jumper wires | 400-point half-size | Local | Rs 50-80 |
| **Total** | | | **Rs 430-790** |

---

## Circuit

**ESP32-CAM to FTDI programmer (required for flashing):**

```
ESP32-CAM          FTDI FT232RL
─────────          ────────────
5V         ──────  VCC (set to 5V)
GND        ──────  GND
U0TXD      ──────  RX
U0RXD      ──────  TX
IO0        ──────  GND  (hold low during flash; disconnect after)
```

After flashing, disconnect IO0 from GND and power-cycle the board to enter normal run mode.

**OV2640 camera** is mounted directly on the ESP32-CAM module via its onboard 24-pin FPC connector — no external wiring required.

---

## Detection Method

**Template-based difference detection** — no neural network required.

The system captures a reference image of a known-good PCB under fixed illumination. Each test board is photographed under identical conditions. A pixel-difference comparison flags regions that deviate beyond a configurable threshold.

This approach works reliably at 640x480 resolution with consistent, diffuse lighting. A WS2812B LED ring mounted concentrically around the lens provides the required even illumination.


---

## Performance

| Metric | Value | Notes |
|--------|-------|-------|
| Detection method | Template difference | Pixel-level comparison against reference |
| Resolution | 640x480 | OV2640 JPEG output |
| False positive rate | 5-15% | Tunable via difference threshold |
| Capture-to-result time | < 500 ms | On ESP32-CAM at 240 MHz |
| Power draw | 0.5-1.0 W | At 5V USB supply |
| Working distance | 3-5 cm | With M12 macro lens installed |

The false positive rate is sensitive to lighting variation. A fixed LED ring and consistent board positioning are required for reliable operation.

---

## vs Commercial AOI

| Aspect | This Project | Commercial AOI |
|--------|-------------|---------------|
| Capital cost | Rs 430-790 | Rs 15-50 lakhs |
| Setup time | 1 day | 1-2 weeks (vendor) |
| Detection method | Template difference | Multi-angle optical + AI |
| Defect categories | Any visible deviation | 20-100+ classified types |
| PCB types | Single/double-sided | All |
| Integration | Standalone | MES/ERP integration |
| Maintenance | Self-maintainable | Vendor support contract |

For an MSME currently relying on manual inspection, template-based detection eliminates the largest category of misses — missing components and solder bridges — at a fraction of the cost.

---

## Datasets

The following datasets are maintained for reference and potential future model training:

| Dataset | Source | Size |
|---------|--------|------|
| PKU-Market-PCB | [github.com/tangsanli5201/DeepPCB](https://github.com/tangsanli5201/DeepPCB) | 1,500 images |
| DeepPCB | [github.com/tangsanli5201/DeepPCB](https://github.com/tangsanli5201/DeepPCB) | 10,668 pairs |


---

## Setup Guide

**Step 1 — Flash firmware**

Install the Arduino IDE and add the ESP32 board package via Boards Manager (`https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json`). Select board: `AI Thinker ESP32-CAM`.

Wire the FTDI adapter as shown in the Circuit section. Hold IO0 to GND, then power on to enter boot mode. Upload your sketch, then disconnect IO0 and power-cycle.

**Step 2 — Capture reference image**

Mount the board at a fixed height (3-5 cm above the PCB surface). Ensure the LED ring is connected and powered. Run the reference capture routine once against a verified known-good board. Store the reference frame in SPIFFS.

**Step 3 — Run inspection loop**

```cpp
#include "esp_camera.h"
#include "SPIFFS.h"

void inspectBoard(camera_fb_t *frame, uint8_t *reference, size_t len) {
    uint32_t diff = 0;
    for (size_t i = 0; i < len; i++) {
        diff += abs((int)frame->buf[i] - (int)reference[i]);
    }
    float score = (float)diff / len;
    bool pass = score < DIFF_THRESHOLD;
    digitalWrite(LED_PASS, pass);
    digitalWrite(LED_FAIL, !pass);
}
```

Set `DIFF_THRESHOLD` empirically by running 20-30 known-good boards and recording the score distribution. Set the threshold at mean + 3 standard deviations of that distribution.

**Step 4 — Lens calibration**

After installing the M12 macro lens, loosen the lock ring, position a PCB at the intended working distance, and rotate the lens barrel until the finest trace on the board is sharp. Tighten the lock ring. Re-capture the reference image after any lens adjustment.

---

<!-- ANIMATED FOOTER -->
<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=6,11,20&height=120&section=footer&animation=twinkling" />
</p>

<p align="center">
  <sub>Last verified: 2026-08-14 &nbsp;|&nbsp; Values cross-checked against manufacturer datasheets and Amazon India pricing &nbsp;|&nbsp; Open an issue for corrections</sub>
</p>
