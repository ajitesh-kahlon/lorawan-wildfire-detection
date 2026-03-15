# 🔥 LoRaWAN Wildfire Detection System

> Long-range RF sensor network for wildfire surveillance in remote regions — 15 km range, sub-millisecond latency, LoRaWAN gateway integration.

**Status:** Capstone project for Bayes Studios · UBC Electrical Engineering · Sep 2025 – May 2026  
**Note:** Source code is under NDA. This repository documents the system architecture, design decisions, and technical results.

---

## Problem

Early wildfire detection in remote, off-grid regions requires sensor networks that can communicate over long distances with minimal infrastructure and power. Commercial solutions are expensive and rely on cellular coverage that doesn't exist in most at-risk areas.

## Solution

A distributed sensor network using **LoRaWAN** for long-range, low-power communication between field sensor nodes and a central gateway. Each node monitors environmental conditions and relays data through a mesh topology to a LoRaWAN gateway connected to a network server.

---

## System Architecture

```
┌─────────────┐     USART (16B/50ms)    ┌──────────────────┐     LoRa PHY 915 MHz     ┌─────────────┐
│  Mesh Sensor │ ──────────────────────▶ │  Nucleo-WL55JC1  │ ─────────────────────▶   │  LoRaWAN    │
│  Head Node   │     <1 ms latency      │  (STM32WL MCU)   │      15 km range         │  Gateway    │
└─────────────┘                         └──────────────────┘                           └──────┬──────┘
                                          │ Dual-core ARM                                     │
                                          │ Cortex-M4 + M0+                                   ▼
                                          │ Sub-GHz radio                              ┌─────────────┐
                                          │                                            │  Network    │
                                                                                       │  Server     │
                                                                                       └─────────────┘
```

## Hardware Platform

| Component | Detail |
|---|---|
| **MCU** | STM32WL55JC — dual-core Cortex-M4 (application) + Cortex-M0+ (radio) |
| **Dev Board** | NUCLEO-WL55JC1 |
| **Radio** | Integrated Sub-GHz LoRa transceiver, 915 MHz ISM band |
| **Mesh Interface** | USART serial link to mesh head node |
| **Sensors** | Environmental sensors on mesh sub-nodes (temperature, humidity, gas) |

## Key Technical Work

### LoRa PHY Configuration & RF Optimization

Configured the LoRa physical layer directly (not using LoRaWAN MAC stack for node-to-gateway link initially) to maximize range in forested terrain:

- **Spreading Factor:** Tuned SF7–SF12 to balance range vs. data rate for the deployment environment
- **Bandwidth:** Tested 125 kHz and 250 kHz configurations
- **Coding Rate:** Evaluated CR 4/5 through 4/8 for error resilience
- **Result:** 25% improvement in reliable communication distance through parameter optimization

### LoRaWAN Gateway Integration

Built middleware layer to handle the full join procedure and uplink transmission:

- Implemented OTAA (Over-The-Air Activation) using DevEUI, JoinEUI, and AppKey
- Abstracted radio driver layer to interface STM32WL hardware with LoRaWAN stack
- Configured RX windows and duty cycle compliance for 915 MHz regulatory requirements

### USART Inter-Node Communication

Established reliable serial data link between mesh head node and the Nucleo gateway node:

- 16-byte packet transmission every 50 ms over USART
- Measured latency: <1 ms per transaction
- Verified timing and signal integrity using logic analyzer captures

### Validation & Testing

Designed and executed 10+ functional test cases covering:

- End-to-end packet delivery (sensor → mesh → gateway → server)
- Latency measurement under varying payload sizes
- Packet loss rate at distance intervals
- RF performance in line-of-sight vs. obstructed conditions

## Project Management

This was a client-facing capstone project for **Bayes Studios**. I led the effort to:

- Translate an ambiguous project scope into a milestone-driven execution roadmap
- Decompose the system into parallel sub-team workstreams (LoRaWAN, mesh networking, validation)
- Run iterative reviews with the client to align deliverables with evolving requirements

## Tools & Technologies

`C` · `STM32CubeIDE` · `STM32 HAL` · `LoRa PHY` · `LoRaWAN` · `USART` · `915 MHz RF` · `Logic Analyzer` · `Git`

## Repository Structure

```
lorawan-wildfire-detection/
├── README.md
└── docs/
    └── (test results, diagrams)
```

## What I Learned

- How to configure and debug RF communication at the physical layer — understanding why spreading factor and coding rate trade-offs matter in real deployments
- The gap between "LoRa works on the bench" and "LoRa works reliably at 15 km in terrain" is enormous, and closing it requires systematic parameter tuning and field testing
- Managing a multi-person embedded project requires clear interface contracts between sub-teams (USART packet format, timing, error handling) defined early

---

*Source code was developed under NDA with Bayes Studios and is not publicly available. This repository contains documentation only.*
