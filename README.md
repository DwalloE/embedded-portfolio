# embedded-portfolio

Eleven embedded systems projects, built in public and in a deliberate sequence: from the
ESP-IDF/FreeRTOS stack I ship professionally on 10,000+ fielded devices, outward through
bare-metal STM32 register work, Zephyr on nRF52840, and embedded Linux. Every repo runs in
your browser or in one command, asserts its claims in CI on every commit, and documents what
the simulator cannot prove.

## The projects

| # | Repo | What it proves | Platform | Run it |
|---|---|---|---|---|
| 01 | [esp32-isr-heartbeat](https://github.com/DwalloE/esp32-isr-heartbeat) | `volatile` via its own disassembly, torn 64-bit reads measured and fixed with a seqlock, ISR discipline with a field failure per rule | ESP32 · ESP-IDF | [browser](https://wokwi.com/projects/474713120227154945) · `make -C test` |
| 02 | [spsc-ring-buffer-esp32](https://github.com/DwalloE/spsc-ring-buffer-esp32) | Lock-free SPSC ring from scratch (no FreeRTOS queue), fed by a raw UART RX ISR — ThreadSanitizer-certified with failure controls, 100% branch coverage gated in CI | ESP32 · ESP-IDF | [browser](https://wokwi.com/projects/474765822779725825) · `make -C test` |
| 03 | [bme280-driver-from-datasheet](https://github.com/DwalloE/bme280-driver-from-datasheet) | An I2C driver from the register map with no vendor library: calibration arithmetic, every bus failure path fault-injected and coverage-gated, CI-captured wire-level VCD traces — plus its own simulated BME280, written from the same datasheet | ESP32 · ESP-IDF | [browser](https://wokwi.com/projects/474771407250609153) · `make -C test` |
| 04 | [freertos-task-architecture](https://github.com/DwalloE/freertos-task-architecture) | Four tasks with deliberate core affinity: firmware-asserted stack high-water margins, a linker-map analysis with a stack-vs-static experiment, priority inversion measured both ways (310 ms vs 40 ms), and a canary-caught overflow CI *requires* | ESP32 · FreeRTOS | [browser](https://wokwi.com/projects/474954927923025921) · `make -C test` |
| 05 | [mqtt-store-and-forward](https://github.com/DwalloE/mqtt-store-and-forward) | Telemetry over deliberately broken connectivity with zero loss and zero duplicates: an NVS store-and-forward queue that survives reboot and torn writes, idempotent IDs, jittered backoff — a chaos suite (broker SIGKILL, mid-PUBLISH cuts, power cuts) asserts the subscriber's own verdict in CI | ESP32 · MQTT | `make -C test chaos` |
| 06 | stm32-baremetal-boot | My own startup.s, vector table, linker script, and clock tree — no HAL, no CMSIS | STM32F103 · bare metal | *next up* |
| 07 | stm32-usart-driver | Interrupt-driven USART by register, with a wire-level failure gallery: framing, overrun, noise | STM32F103 · bare metal | *planned* |
| 08 | iot-power-budget-model | Measured phase timings + datasheet currents → defensible battery-life arithmetic, and what an ammeter gets wrong | ESP32 + Python | *planned* |
| 09 | zephyr-nrf52840-sensor-node | Project 03's driver as a proper out-of-tree Zephyr module: devicetree binding, Kconfig, Twister + Renode tests | nRF52840 · Zephyr | *planned* |
| 10 | embedded-linux-gateway | A bootable Buildroot image with my own kernel character driver and a packaged gateway daemon | ARM · Linux/QEMU | *planned* |
| 11 | fleet-mrv-signed-telemetry | Capstone: provisioning, authenticated anti-replay telemetry, A/B OTA with rollback, staged rollout — my actual domain, made inspectable | device → cloud | *planned* |

## How to read these repos

Each one follows the same contract: a GIF of it running, a browser link, what it
demonstrates in interview vocabulary, a **bug gallery** (what goes wrong when you get it
wrong, with evidence — the section that separates working code from understood code), how CI
asserts it, and honest limits.

Two habits worth noticing, because they are the actual product:

- **Tests prove they can detect failure before claiming success.** Project 01 first shows
  the harness catching a torn read, then shows the seqlock preventing it; project 02 first
  makes ThreadSanitizer flag a barrier-stripped buffer and two abusive writers, then lets
  the real one claim a clean run; project 05 runs one chaos pass with dedup disabled (the
  duplicate must reach the consumer) and one boot with the CRC check stubbed (the torn
  record must sail through) before trusting either green. A safety net that was never
  proven able to fire is decoration.
- **Honest limits are stated, not implied.** A simulator has no analog behaviour, measures
  no current, and x86 ThreadSanitizer certifies a memory-ordering discipline, not Xtensa
  silicon. Each README says exactly what its evidence covers and where bench work begins.

## Why this sequence

The list is ordered to close specific gaps in a specific order: ISR and driver-level C on
the stack I already ship (01–05, each one interview-depth on a question actually asked),
then the bare-metal register work an ESP-IDF career abstracts away (06–08), then the
ecosystems hiring is moving toward (Zephyr, 09; embedded Linux, 10), and finally a capstone
(11) that only someone running a real fleet can write.

Everything here is simulator-first by necessity and says so. What a simulator cannot show —
analog behaviour, real timing margins, current draw, RF — I do on fielded hardware at my day
job; these repos exist to make the code-depth side public and reviewable.

MIT licensed.
