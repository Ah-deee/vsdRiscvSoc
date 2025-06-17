# Checklist questions

# week2 notes

## 1. Four Major Sub-Modules Instantiated Inside `caravel`

Based on the `caravel.v` top module, the four major sub-modules instantiated inside `caravel` are:

- **Management SoC**: This is typically the main processor or controller block responsible for managing the chip's core functions.
- **User Project Area**: This block is reserved for user-defined logic and is often isolated from critical management functions.
- **Housekeeping SPI**: This module handles housekeeping tasks, including SPI communication for configuration and monitoring.
- **Power-On Reset (POR) Block**: This ensures the chip starts in a known state by generating a reset signal during power-up.


## 2. Signals Crossing the “Management Protect” Boundary

The “Management Protect” boundary is a security or isolation barrier between the management SoC and the user project area. The signals that typically cross this boundary include:

- **Wishbone Bus Signals**: These are used for communication between the management SoC and user project logic.
- **Reset Signals**: Allowing the management block to reset user logic.
- **Clock Signals**: Providing a clock from the management domain to the user area.
- **GPIO and Interrupt Lines**: For user logic to interact with external pins or signal interrupts to the management SoC.


## 3. Where Reset and Clock First Get Synchronized in the Hierarchy

Within the `caravel` hierarchy:

- **Reset Synchronization**: The reset signal is first synchronized in the Power-On Reset (POR) block, which generates a clean, debounced reset for the rest of the design.
- **Clock Synchronization**: The primary clock is first buffered and distributed at the top level of the hierarchy, often immediately after entering the `caravel` module, before being routed to sub-modules like the management SoC and user project area.

  ---

  Caravel block diagram:

  ![Image](https://github.com/user-attachments/assets/e3ecca94-0c43-4235-9f19-c5f1eed1865f)

