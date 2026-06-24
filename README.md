# STM32 DMA UART Daisy-Chain Network

## Overview
This repository contains the implementation for a multi-node communication network using STM32 microcontrollers. The system uses a daisy-chain (ring) topology over standard UART lines, fully driven by Direct Memory Access (DMA) to ensure zero CPU blocking during data transmission and reception.

## Technical Features
* **Zero-Blocking Architecture:** Utilizes STM32's `HAL_UARTEx_ReceiveToIdle_DMA` to handle variable-length packets without interrupting the main execution loop.
* **Dynamic Auto-Addressing:** Nodes automatically assign themselves a sequential ID (e.g., `_43_0`, `_43_1`) by counting delimiter characters in the received payload.
* **Hardware XOR Checksum:** Implements a custom XOR checksum calculation appended to the end of each payload. If a node detects a checksum mismatch, it halts data propagation and triggers a status indicator (LD3 solid ON).
* **Head/Tail Routing Logic:** The system dynamically distinguishes between the "Head" node (connected to the PC) and middle nodes. The Head node routes data from the PC down the chain, and catches the returning payload to transmit back to the PC.

## Hardware Configuration & Pinout
Ensure all boards share a common Ground (GND). Connect the TX of one board to the RX of the next board in the chain.

| Peripheral | Function | Standard Pins |
| :--- | :--- | :--- |
| **USART2** | PC Interface (USB-to-TTL) | TX: PA2 / RX: PA3 |
| **USART1** | Inter-board Daisy-Chain | TX: PA9 / RX: PA10 |
| **GPIO Output**| Error/Status Indicator | LD3 (e.g., PB3 or PA5) |

## System Data Flow
1. **Initialization:** The PC sends a starting token via `USART2` to the first board in the chain (Board 0).
2. **Token Injection:** Board 0 intercepts the packet, sets its state to `isHead`, appends its suffix (`_43_0`), calculates the XOR checksum, and transmits the payload via `USART1` to the next board.
3. **Relay & Validation:** Board 1 receives the data on `USART1`. It recalculates the XOR checksum to verify data integrity. If valid, it replaces the previous suffix with its own (`_43_1`), calculates a new checksum, and forwards it to the next board.
4. **Loop Completion:** Once the data passes through all nodes and returns to Board 0's `USART1`, Board 0 identifies the completed loop and transmits the final data package back to the PC via `USART2`.

## Repository Contents
* `main.c`: Contains the core application logic, DMA callback functions (`HAL_UARTEx_RxEventCallback`), and checksum algorithms.
* `.zip` / `.ioc` files: Complete STM32CubeIDE project setup parameters.
