# Physical Installation Guide

Enervent Pingvin Kotilämpö eAir — Waveshare ESP32-S3-RS485-CAN

---

## Parts & Tools Needed

| Item | Notes |
|---|---|
| Waveshare ESP32-S3-RS485-CAN board | The onboard RS485 transceiver is pre-wired — no extra module needed |
| USB-C cable | For initial flashing only |
| 5 V USB power supply or USB-C charger | ≥ 500 mA; used for permanent power after flashing |
| 2-conductor twisted-pair cable | Standard alarm/doorbell cable works; shielded is better |
| Small flat-head screwdriver | For the Pingvin terminal block |
| Multimeter | Optional but helpful for verifying polarity |

---

## Step 1 — Flash the Firmware First

Flash the device **before** mounting it inside the unit. It is much easier to connect via USB on your desk.

```bash
cp secrets.yaml.example secrets.yaml   # fill in your details
esphome run pingvin_ventilation.yaml
```

After the first successful flash, all future updates can be done over Wi-Fi (OTA).

---

## Step 2 — Locate the Modbus Connector on the Pingvin

1. **Switch off power to the Pingvin unit** at the circuit breaker before opening the cover.
2. Open the front panel of the Pingvin unit (two screws at the bottom).
3. The motherboard (MD board) is visible on the right side of the unit.
4. Look for the RS485 / Modbus terminal block — it is typically labeled **A / B** or **RS485** near the communication connectors along the edge of the board.

> **Caution:** The motherboard also carries mains voltage (230 V) on other connectors. Only touch the low-voltage RS485 terminals. When in doubt, consult the Enervent installation manual for your specific model.

---

## Step 3 — Prepare the Cable

Cut your twisted-pair cable to the required length (from the Pingvin to wherever you will mount the ESP32 board).

- Strip about **6 mm** of insulation from each conductor at both ends.
- Twist or tin the bare ends so they do not fray in the terminal block.
- If using shielded cable, connect the shield to GND **at one end only** (preferably the Pingvin end) to avoid ground loops.

---

## Step 4 — Connect to the Pingvin Motherboard

| Cable conductor | Pingvin terminal |
|---|---|
| Conductor 1 (e.g. white) | **A** (non-inverting, sometimes marked **+**) |
| Conductor 2 (e.g. black) | **B** (inverting, sometimes marked **−**) |
| Shield / GND (if shielded) | **GND** terminal next to A/B (if present) |

Insert each wire into the terminal, tighten the screw firmly, and give a gentle tug to confirm it is held.

---

## Step 5 — Connect to the Waveshare ESP32 Board

The Waveshare ESP32-S3-RS485-CAN board exposes the RS485 bus on a screw-terminal block labelled **A** and **B** (and GND).

| Cable conductor | Waveshare terminal |
|---|---|
| Conductor 1 (white, from Pingvin A) | **A** |
| Conductor 2 (black, from Pingvin B) | **B** |
| Shield / GND (if used) | **GND** |

```
Pingvin motherboard          Waveshare ESP32-S3 board
────────────────────         ──────────────────────────
A  (non-inverting)  ──────►  A
B  (inverting)      ──────►  B
GND / Shield        ──────►  GND  (one end only)
```

> **Polarity matters on RS485.** If the connection appears to work but returns garbage data or no response, swap the A and B wires — it is the most common first mistake.

---

## Step 6 — Bus Termination

RS485 buses require a **120 Ω termination resistor at each physical end** of the cable to prevent signal reflections.

| Scenario | Action |
|---|---|
| Only the ESP32 and the Pingvin on the bus (point-to-point, most common) | Enable the 120 Ω jumper on the Waveshare board. The Pingvin has its own internal termination. |
| Other RS485 devices are also connected (e.g. room panels on the same bus segment) | Only the two devices at the **physical ends** of the cable get termination. Do not enable the jumper on intermediate nodes. |

**To enable termination on the Waveshare board:**
Locate the solder jumper or pin header labelled **120R** or **Term** on the board and close/enable it per the board's silkscreen or Waveshare wiki.

---

## Step 7 — Power the ESP32

After wiring, connect a 5 V USB-C power supply to the Waveshare board. You can:

- Route a USB-C cable from a nearby USB charger or USB port on your router/NAS.
- Mount the board inside the Pingvin enclosure if space allows — the unit has a 12 V or 24 V auxiliary supply on the motherboard, but you would need a small DC-DC converter to step down to 5 V. **The simplest option is an external USB charger.**

---

## Step 8 — Verify the Connection

1. Restore power to the Pingvin at the circuit breaker.
2. Watch the ESPHome device logs (via `esphome logs pingvin_ventilation.yaml` or the Home Assistant ESPHome add-on).
3. A successful Modbus session looks like:

```
[D] Modbus component: Sending 8 bytes
[D] ModbusController: All sensors updated
[D] sensor: 'Supply Air Temperature' -> 21.4 °C
```

4. If you see repeated `No response from slave` errors:
   - Double-check A/B polarity (swap if needed).
   - Verify the Pingvin Modbus address is 1 (default) in its settings menu.
   - Confirm baud rate is 19200 in the Pingvin settings (Communication → Modbus speed).

---

## Mounting the ESP32 Board

- Keep the board away from mains wiring inside the Pingvin — maintain at least a few centimetres of separation.
- The board has mounting holes; use nylon standoffs if attaching to a metal surface to avoid shorts.
- Ensure Wi-Fi signal can reach the board. Metal enclosures can attenuate the signal; you may need to route the antenna outside the unit or mount the board on the exterior of the enclosure.

---

## Quick Reference

| Parameter | Value |
|---|---|
| Modbus protocol | RTU |
| Baud rate | 19200 bps |
| Framing | 8N1 (8 data bits, no parity, 1 stop bit) |
| Slave address | 1 |
| RS485 A pin (ESP32) | GPIO17 via onboard transceiver |
| RS485 B pin (ESP32) | GPIO18 via onboard transceiver |
| Flow control pin | GPIO21 (DE/RE) |
| ESP32 supply voltage | 5 V via USB-C |
| Termination resistor | 120 Ω at each end of the bus |
