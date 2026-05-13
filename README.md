# Enervent Pingvin Kotilämpö — ESPHome Integration

ESPHome configuration for integrating the **Enervent Pingvin Kotilämpö eAir** heat-recovery ventilation unit into Home Assistant via Modbus RTU over RS485.

## Hardware

| Component | Details |
|---|---|
| Microcontroller | [Waveshare ESP32-S3-RS485-CAN](https://www.waveshare.com/wiki/ESP32-S3-RS485-CAN) |
| Interface | RS485 (onboard transceiver, internally wired) |
| Protocol | Modbus RTU |
| Ventilation unit | Enervent Pingvin Kotilämpö eAir (MD board, firmware ≥ 1.14) |

### Onboard RS485 pin mapping (Waveshare board)

| Signal | GPIO |
|---|---|
| TX | GPIO17 |
| RX | GPIO18 |
| DE/RE (flow control) | GPIO21 |

### Modbus parameters

| Parameter | Value |
|---|---|
| Speed | 19200 bps |
| Framing | 8N1 |
| Slave address | 1 (Pingvin default) |

## Wiring

Connect the Waveshare board's RS485 terminals to the Pingvin motherboard's Modbus connector:

```
Waveshare board          Pingvin motherboard
──────────────────       ──────────────────────
A  (non-inverting)  ───► A  (non-inverting / +)
B  (inverting)      ───► B  (inverting / -)
GND                 ───► GND  (one end only)
```

**Termination:** RS485 requires a 120 Ω resistor at each physical end of the bus.
The Pingvin unit has built-in termination. Enable the onboard 120 Ω jumper on the Waveshare board if it is the only other node on the bus (point-to-point).

## Features

### Sensors (read-only)
- Room temperatures: Panel 1, Panel 2, Average
- Outside air temperature + 24 h moving average
- Supply air temperature (after HRC, after heater)
- Extract & exhaust air temperatures
- Supply & exhaust fan speeds (%)
- Extract air humidity + 48 h average
- Supply & exhaust filter pressure differences (Pa)
- Heat recovery efficiency — supply & exhaust sides (%)
- Temperature controller output
- Temperature controller mode (text: Idle / Cooling / Heat Recovery / Heating / …)

### Status binary sensors
- 9 status flags from the `HREG_MODE` bitfield (max heating/cooling, away, boosts, SNC, defrost…)
- Heat recovery wheel running
- Supply air heater element active
- Alarm A / Alarm B active

### Controls (writable)
| Entity type | Name |
|---|---|
| `number` | Supply Air Temperature Setpoint (10–30 °C, step 0.5) |
| `switch` | Stop Machine |
| `switch` | Away Mode / Away Long Mode |
| `switch` | Max Heating / Max Cooling |
| `switch` | CO₂ Boosting / Humidity Boosting |
| `switch` | Manual Boost (100%) / Temperature Boost |
| `switch` | Summer Night Cooling |
| `switch` | Heating / Cooling in Away Mode |
| `switch` | Temperature Decrease |
| `switch` | Eco Mode / Silent Mode |
| `switch` | Cooling Enabled / Heating Enabled |
| `switch` | HRC Defrosting Enabled |

## Setup

### 1. Clone and configure secrets

```bash
git clone git@github.com:simopaso/EnerventESP.git
cd EnerventESP
cp secrets.yaml.example secrets.yaml
# Edit secrets.yaml with your Wi-Fi credentials and API key
```

### 2. Generate an API encryption key

```bash
python3 -c "import base64, os; print(base64.b64encode(os.urandom(32)).decode())"
```

Paste the output into `secrets.yaml` as `api_encryption_key`.

### 3. Flash

```bash
esphome run pingvin_ventilation.yaml
```

After the first OTA-capable flash, subsequent updates can be done wirelessly.

### 4. Add to Home Assistant

The device will appear automatically via the ESPHome integration once it is on the same network.

## Register reference

`eAirMD-modbus-register-list-public.csv` — official Modbus register map from Enervent (holding registers and coils), conforming to MD firmware 1.14 and later.

## License

MIT
