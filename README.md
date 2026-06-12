# HC-05

Copy-Item -Path "C:\Users\julie\STM32CubeIDE\workspace_1.18.1\bluetot" -Destination "C:\Users\julie\Documents\Github\maxime\bluetot" -Recurse -Container

# STM32 Bluetooth RC Car — HC-05 UART Link

Bluetooth communication between an STM32 Nucleo F401RE and a PC/controller
via an HC-05 module, using interrupt-driven UART reception.

## Hardware

| HC-05 | Nucleo F401RE |
|-------|---------------|
| VCC   | 5V            |
| GND   | GND           |
| TX    | PA10 (USART1 RX) |
| RX    | PA9 (USART1 TX)  |
| STATE | PA5           |

No level shifting needed: the STM32 TX and HC-05 outputs are both 3.3V logic.
PA5 is shared with the onboard LD2, which lights up when a Bluetooth
connection is active.

## CubeMX Configuration

- **USART1**: Asynchronous, 9600 baud, 8N1 (HC-05 data mode)
- **NVIC**: USART1 global interrupt enabled
- **USART2**: Asynchronous, 115200 baud (ST-Link VCP, printf debug)
- **PA5**: GPIO Input (HC-05 STATE)

## Firmware Overview

- `printf` is redirected to USART2 via `_write()`
- Reception is interrupt-driven: `HAL_UART_Receive_IT()` (1 byte),
  re-armed in `HAL_UART_RxCpltCallback()`
- Main loop polls the STATE pin, prints status and received bytes to the
  debug port, and echoes each byte back over Bluetooth

## PC Connection

> The HC-05 uses Bluetooth Classic (SPP) — it will **not** work with iPhones.

1. Pair "HC-05" in Windows Bluetooth settings (PIN `1234` or `0000`)
2. Find the **outgoing** COM port (More Bluetooth settings → COM Ports)
3. PuTTY #1: ST-Link COM port @ 115200 → debug output
4. PuTTY #2: Bluetooth outgoing COM port @ 9600 → data link
   (connection establishes when the port opens; the HC-05 blink slows)
5. Type in PuTTY #2 → `Received: x` in #1, `Echo: x` back in #2

## Build

Open the project in STM32CubeIDE and flash via the onboard ST-Link.



----------------------------------------------------------