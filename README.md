# ESP32 + GTW-08 — Remeha / Intergas / Baxi / Brötje heat pump

[ESPHome](https://esphome.io/) firmware to read and control a heat pump (or Ace boiler) through the **GTW-08 Modbus RTU gateway**, from [Home Assistant](https://www.home-assistant.io/).

<p align="center">
  <img src="images/hardware.jpg" alt="ESP32, MAX485 module and GTW-08 gateway" width="720">
</p>

The config exposes temperatures, pressure, heat-pump / DHW / cooling state, setpoints, heating curve, and switches (heating, DHW, cooling).

---

## What it is for

Without this gateway the heat pump only talks through the vendor cloud. **GTW-08** exposes **Modbus RTU (RS-485)**. The ESP32 bridges that bus to Wi‑Fi and the native Home Assistant API (local, no cloud).

Typical uses:

- dashboard (flow / return, delta T, outdoor, DHW tank)
- cut heating / DHW from tariff, occupancy or PV
- alerts for low water pressure, faults, service required
- room setpoint, heating-curve slope, DHW setpoint from HA

Works with **Ace** products (T-Control / S-Control) that accept a GTW-08: Remeha Elga Ace, Mercuria Ace, Eria Tower, Intergas, Baxi, Brötje, De Dietrich, and other BDR Thermea siblings.

---

## Hardware

| Part | Role | Link |
| --- | --- | --- |
| **GTW-08** (Remeha 7721982 / ML638) | Internal bus → Modbus RTU | [Product page](https://www.alternative-haustechnik.de/remeha-schnittstelle-modbus-rtu-gateway-gtw-08/7721982) · [Amazon](https://www.amazon.co.uk/Remeha-Modbus-interface-Gateway-GTW-08/dp/B0971GM23N) |
| **ESP32 DevKit** | Wi‑Fi + UART | any 30-pin ESP32 |
| **MAX485 / TTL-RS485** | 3.3 V UART → RS-485 | use a 3.3 V module |
| 5 V USB supply | ESP32 | — |
| Twisted pair | Modbus A / B | alarm / bus cable, 2 wires |

Vendor doc: [GTW-08 configuration guide (PDF)](https://tools.remeha.nl/wp-content/uploads/sites/11/2024/07/configuratiehandleiding-GTW-08.pdf).

Related projects:

- [Imanol82 — Baxi/Dietrich/Remeha → ESP32](https://github.com/Imanol82/Baxi-Dietrich-Remeha-to-Home-Assistant-with-an-ESP32)
- [houthacker/remeha-modbus](https://github.com/houthacker/remeha-modbus) (Modbus TCP, different path)

<p align="center">
  <img src="images/gtw08-install.jpg" alt="GTW-08 installed in the heat-pump cabinet" width="480">
</p>

*Example of the module installed in the heat-pump cabinet (credit: Imanol82 project).*

---

## Wiring

Modbus settings used here:

| Parameter | Value |
| --- | --- |
| Speed | **9600** 8N1 |
| Slave | **0x64 (100)** — GTW-08 rotary switch |
| ESP32 TX | **GPIO17** → DI / TX on the MAX485 |
| ESP32 RX | **GPIO16** → RO / RX on the MAX485 |
| MAX485 A / B | GTW-08 A / B terminals |
| MAX485 VCC / GND | ESP32 3.3 V and GND |
| DE+RE | often tied to 3.3 V (always transmit) or to a GPIO if the module needs it |

```
Heat pump Ace  --(internal bus)-->  GTW-08  --RS-485 A/B-->  MAX485  --UART-->  ESP32  --Wi-Fi-->  Home Assistant
```

<p align="center">
  <img src="images/wiring-reference.png" alt="ESP32 MAX485 GTW-08 wiring diagram" width="640">
</p>

*Reference diagram (Imanol82 project). This repo uses TX=GPIO17, RX=GPIO16, 9600 baud, slave 100.*

If nothing answers: swap **A and B**, check the address (rotary switch = 100), and confirm the GTW-08 is powered from the heat-pump internal bus.

---

## ESPHome setup

1. Install [ESPHome](https://esphome.io/) (HA add-on or CLI).
2. Copy `esp32-pac.yaml` and `secrets.yaml.example` → `secrets.yaml`.
3. Fill in Wi‑Fi, OTA password, fallback hotspot, and an API key:

```bash
openssl rand -base64 32
```

4. First flash over USB, then OTA.

```bash
esphome run esp32-pac.yaml
```

5. In Home Assistant: **Settings → Devices & services → Add → ESPHome** (auto-discovery).

---

## Exposed entities

**Sensors:** output %, outdoor temperature, heat-pump flow / return, delta T, DHW tank, DHW setpoint, room temperature, heating setpoint, water pressure, pump speed, error code.

**Binary sensors:** heat pump on, backup 1/2, DHW backup, service, low pressure, fault, CH / DHW / cooling active, pumps.

**Controls:** enable heating / DHW / cooling; room setpoint; curve slope and foot point; DHW setpoint; hysteresis; heating / DHW modes (schedule, manual, frost protection); outdoor cut-off temperatures.

Modbus addresses follow the GTW-08 holding-register table. Only change slave / baud if your rotary switch is not set to 100.

---

## License

MIT — see `LICENSE`.
GTW-08, Remeha, Intergas, Baxi and Brötje are trademarks of their owners.
