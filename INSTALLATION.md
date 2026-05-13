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
| Small flat-head screwdriver | For the Pingvin terminal block screws |
| Multimeter | Optional but helpful for verifying polarity |

---

## Step 1 — Flash the Firmware First

Flash the device **before** mounting it near the unit. It is much easier to connect via USB on your desk.

```bash
cp secrets.yaml.example secrets.yaml   # fill in your details
esphome run pingvin_ventilation.yaml
```

After the first successful flash, all future updates can be done over Wi-Fi (OTA).

---

## Step 2 — Open the Pingvin Electronics Enclosure

The Pingvin unit has two distinct sections inside its cabinet:

- **Air handling section** — contains the fans, heat exchanger and filters.
- **Electronics enclosure** — a separate sealed box containing the motherboard (MD board), control electronics, and the RS485 Modbus connector.

The RS485 connector is on the **motherboard inside the electronics enclosure**. You need to open the electronics enclosure, not just the air handling section.

**Before opening:**

1. **Switch off the Pingvin at the circuit breaker** — not just via the control panel. The motherboard carries 230 V mains on some connectors; full power isolation is required.
2. Wait a moment for capacitors to discharge.
3. Open the electronics enclosure per the Enervent installation manual for your model (typically screws on the enclosure lid).

> **Caution:** Only handle the low-voltage RS485/Modbus terminals on the motherboard. The mains (230 V) connectors are on the same board. If you are unsure which terminals are which, consult the Enervent eAir installation guide before touching anything.

---

## Step 3 — Locate the RS485 Connector on the Motherboard

The RS485 Modbus connector on the MD motherboard is a **pluggable terminal block** — it consists of two parts:

- A **fixed header** soldered to the board.
- A **removable plug** that you pull out, wire up, and push back in.

Look for the connector labelled **A / B** or **RS485** along the edge of the motherboard. Pull the removable plug straight out of its header before attaching your cable — it is easier to work with off the board.

---

## Step 4 — Prepare the Cable

Cut your twisted-pair cable to the required length (from the Pingvin to wherever you will mount the ESP32 board).

- Strip about **6 mm** of insulation from each conductor at both ends.
- Twist or tin the bare ends so they do not fray in the terminal block.
- If using shielded cable, connect the shield to GND **at one end only** (preferably the Pingvin end) to avoid ground loops.

---

## Step 5 — Wire the Pingvin Connector Plug

Insert the conductors into the **removable plug** (pulled out in Step 3) and tighten the screws firmly. Give each wire a gentle tug to confirm it is held.

| Cable conductor | Plug terminal |
|---|---|
| Conductor 1 (e.g. white) | **A** (non-inverting, sometimes marked **+**) |
| Conductor 2 (e.g. black) | **B** (inverting, sometimes marked **−**) |
| Shield / GND (if shielded) | **GND** terminal (if present on plug) |

> **Do not cross the wires.** A connects to A and B connects to B at both ends of the cable. Crossing them is the most common mistake and produces no response from the device.

Once wired, push the plug back into the fixed header on the motherboard until it clicks or seats fully.

---

## Step 6 — Connect to the Waveshare ESP32 Board

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

> **If the connection produces no data or garbage values**, the most likely cause is swapped A/B polarity. Swap the two conductors at either end and try again.

---

## Step 7 — Bus Termination

RS485 buses require a **120 Ω termination resistor at each physical end** of the cable to prevent signal reflections.

| Scenario | Action |
|---|---|
| Only the ESP32 and the Pingvin on the bus (point-to-point, most common) | Enable the 120 Ω jumper on the Waveshare board. The Pingvin has its own internal termination. |
| Other RS485 devices also on the same bus segment | Only the two devices at the **physical ends** of the cable get termination. Do not enable the jumper on intermediate nodes. |

**To enable termination on the Waveshare board:**
Locate the solder jumper or pin header labelled **120R** or **Term** and close/enable it per the board's silkscreen or Waveshare wiki.

---

## Step 8 — Power the ESP32

Connect a 5 V USB-C power supply to the Waveshare board:

- A standard USB-C phone charger next to the unit is the simplest option.
- The Pingvin motherboard has auxiliary low-voltage power, but using it requires a DC-DC converter and is not recommended for a first installation.

---

## Step 9 — Verify the Connection

1. Restore power to the Pingvin at the circuit breaker.
2. Watch the ESPHome device logs via `esphome logs pingvin_ventilation.yaml` or the Home Assistant ESPHome add-on.
3. A successful Modbus session looks like:

```
[D] Modbus component: Sending 8 bytes
[D] ModbusController: All sensors updated
[D] sensor: 'Supply Air Temperature' -> 21.4 °C
```

4. If you see repeated `No response from slave` errors:
   - Swap A and B wires (polarity).
   - Confirm Modbus slave address is 1 in the Pingvin settings menu.
   - Confirm baud rate is 19200 in Pingvin settings (Communication → Modbus speed).
   - Check that the removable connector plug is fully seated in the board header.

---

## Mounting the ESP32 Board

- Keep the board away from mains wiring — maintain at least a few centimetres of separation from any 230 V cables.
- The board has mounting holes; use nylon standoffs if attaching to a metal surface to avoid shorts.
- Metal enclosures can attenuate the Wi-Fi signal significantly. If signal is weak, mount the board on the **exterior** of the electronics enclosure, or route just the RS485 cable through a small hole.

---

## Known Limitations of the Modbus Interface

These were confirmed by real-world testing on the Pingvin eAir (Rankinen, OAMK thesis 2023):

- **Most holding registers are effectively read-only.** Even though Modbus protocol allows writing them and the device acknowledges the write, most register values revert immediately to their previous state. The manufacturer has restricted external control via RS485.
- **The temperature setpoint (hreg 135) is the only holding register confirmed writable** from external Modbus.
- **Fan speed cannot be set directly.** Only operating modes (coils) can be changed, which indirectly affects fan speed.
- **Some coils are mutually exclusive** — only one mode-state coil can be active at a time (e.g. Away, Max Heating, Max Cooling). The Pingvin firmware enforces this; if you force one on, any conflicting modes are automatically cleared.
- **Mode changes time out automatically.** The Pingvin reverts boosted modes (Manual Boost, Max Heating, etc.) after a built-in timer expires. The timer cannot be disabled via Modbus.

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
| Connector type on Pingvin MB | Pluggable screw terminal block (removable plug) |
| Confirmed writable register | hreg 135 — supply air temperature setpoint |
