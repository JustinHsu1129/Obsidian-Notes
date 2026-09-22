# Bit-Banging Communication Protocol Rates & Technical Limits

A reference guide summarizing realistic bit-banging throughput, hardware limits, clocking topologies, and physical bus bottlenecks across common embedded protocols.

| Protocol | Typical Bit-Bang Range | Hardware Spec Limits | Clocking Model | Line Drive Type | Primary Bottleneck |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **UART** | **9.6 kbps – 115.2 kbps** | ~10–20 Mbps (ASIC transceiver) | Asynchronous (no clock line) | Push-Pull | Timing jitter, cycle drift, and interrupt latency; lack of a shared reference clock. |
| **I2C** | **10 kbps – 400 kbps** | 100 kHz (Standard)<br>400 kHz (Fast)<br>1 MHz (Fast+)<br>3.4 MHz (High-Speed) | Synchronous (Master-driven SCL) | Open-Drain (requires pull-up resistors) | Bus capacitance and pull-up $RC$ rise times; bidirectional pin tri-stating overhead for ACK/NACK. |
| **SPI** | **100 kbps – 10 Mbps+** | 50–100 Mbps+ | Synchronous (Master-driven SCK) | Push-Pull (active drive high/low) | Raw instruction execution latency and GPIO register write speed. Fully tolerant of clock jitter. |
| **JTAG** | **100 kbps – 6 Mbps** | 10–50 MHz | Synchronous (TCK edge-triggered FSM) | Push-Pull | Host USB-to-bridge turnaround latency (when bit-banged via FTDI/GPIO adapters). |
| **USB 1.1 (Low-Speed)** | **1.5 Mbps** (fixed) | 1.5 Mbps | Asynchronous (differential D+/D-) | Push-Pull (with termination) | Strict cycle-accurate instruction budgeting; real-time NRZI encoding and bit-stuffing overhead. |
| **USB 1.1 (Full-Speed)** | **12 Mbps** (rare) | 12 Mbps | Asynchronous (differential D+/D-) | Push-Pull (with termination) | Exceeds pure software GPIO loop speeds; requires programmable state machines (e.g., RP2040 PIO). |
| **USB 2.0 (High-Speed)** | **Impossible** | 480 Mbps | Asynchronous (differential) | Analog Transceiver (PHY) | Slew rate limits, impedance matching, and analog clock-data recovery (CDR) requirements. |

---

## Technical Summary of Limiting Factors

### 1. Synchronous vs. Asynchronous Demands
* **Synchronous (SPI, I2C, JTAG):** The presence of an explicit clock line ($SCLK$, $TCK$) makes the bus forgiving of software delays or intermittent interrupts. The clock can be slowed down or held indefinitely (unless the target slave implements a watchdog timeout).
* **Asynchronous (UART, USB):** Data is sampled based on precise time intervals relative to a synchronization edge (start bit or packet sync). Any software jitter exceeding approximately $\pm2.5\%$ to $\pm5\%$ of a bit period results in framing errors or packet rejection.

### 2. Physical Output Stages
* **Push-Pull (SPI, UART):** Actively switches between $V_{CC}$ and $GND$ using complementary transistors. Rise and fall times are dictated by the microcontroller output pad driver (typically $<5\text{ ns}$).
* **Open-Drain (I2C):** Active low driving only; line pull-up relies on passive external resistors ($1\text{ k}\Omega - 10\text{ k}\Omega$). The rising edge follows an exponential $RC$ charging curve:
  $$t_r \approx 0.8473 \times R_p \times C_b$$
  Software loops must insert explicit pauses to satisfy setup times ($t_{SU:DAT}$) before pulsing the clock.