# README for GPIO State Monitoring

## Overview

This project implements a GPIO (General Purpose Input/Output) state monitoring system using an interrupt service routine (ISR) to sample digital input pin states. The system maintains a consistent state for each pin over a specified number of samples to filter out noise and ensure reliable readings.

## Features

- Monitors up to 8 digital input pins.
- Implements a consistency check to avoid false readings due to noise.
- Uses a bitmask to represent the state of each pin.

## Definitions

- **NUM_INPUTS**: Defines the number of digital input pins to monitor (set to 8).
- **CONSISTENCY_CHECK**: Defines the number of consecutive samples required for a pin state to be considered stable (set to 10).

## Global Variables

- `volatile uint8_t g_AppDIpinSts`: A volatile variable that holds the application-level state of the digital input pins. Each bit corresponds to the state of a pin (1 for high, 0 for low).
- `uint8_t g_ReadDIpinSts`: A variable that stores the current readings from the digital input pins.

## Structs

### GpioState

```c
struct GpioState {
  uint8_t consistent_count; // Counts consecutive samples of the same pin state
  uint8_t last_pin_state;   // Holds the last known state of the pin
};
```

- `consistent_count`: A counter that increments when the current pin state matches the last known state.
- `last_pin_state`: The last recorded state of the pin.

## Array

- `volatile struct GpioState pinStatus[NUM_INPUTS]`: An array of `GpioState` structures, one for each input pin, to keep track of the state and consistency count.

## Function: ISR_DIsampling

### Signature

```c
int ISR_DIsampling();
```

### Description

This function is called in response to an interrupt signal, typically triggered by a timer or an external event. It performs the following tasks:

1. Reads the current states of the digital input pins.
2. Iterates through each pin:
   - Checks if the current state is the same as the last recorded state.
   - Increments the `consistent_count` if the state is consistent; resets it if not.
   - If the `consistent_count` reaches the defined `CONSISTENCY_CHECK`, updates the `g_AppDIpinSts` to reflect the stable state.
   - Resets the `consistent_count` after updating the application state.
   - Updates the `last_pin_state` with the current state.

### Return Value

- Returns `0` upon successful completion.

## Usage

1. Include this code in your embedded system project.
2. Set up an appropriate interrupt mechanism to call `ISR_DIsampling()` regularly (e.g., using a timer or external interrupt).
3. Use `g_AppDIpinSts` to access the stable states of the digital input pins.

## Example

To read the state of the pins after the ISR has been executed, you can access `g_AppDIpinSts` as follows:

```c
if (g_AppDIpinSts & (1 << pin_index)) {
    // Pin is HIGH
} else {
    // Pin is LOW
}
```

## Conclusion

This GPIO state monitoring system is designed to provide reliable readings from digital input pins by implementing a consistency check mechanism. It is suitable for applications where noise and transient states may affect the accuracy of pin readings.
