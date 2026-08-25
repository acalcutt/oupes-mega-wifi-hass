# OUPES Mega WiFi — Home Assistant Integration

Custom Home Assistant integration for **OUPES Mega** power stations over
**WiFi** — fully local, no cloud dependency required.

> The **BLE** integration lives in its own repository:
> [acalcutt/oupes-mega-hass](https://github.com/acalcutt/oupes-mega-hass).
> HACS requires one integration per repository, so the two halves of this
> project were split. The BLE and WiFi integrations can run side by side —
> they use independent communication channels and create separate device and
> entity sets.

---

## What it does

**Merged WiFi integration** — intercepts the device's outbound connection to the
OUPES cloud broker, serves it locally, and exposes WiFi telemetry as HA entities.

- TCP broker proxy (port 8896) — device connects here instead of the cloud
- HTTP API emulator (port 8897) — handles Cleanergy app REST calls
- SiBo HTTPS stub (port 8898) — prevents app "token error" login loops
- Push-based telemetry (no polling) — entities update in real time
- Same sensors as BLE (battery, power, temperature, runtime)
- Output control via TCP commands
- Requires firewall NAT rules to redirect device/app traffic

**[Full documentation →](custom_components/oupes_mega_wifi/README.md)**

---

## Installation

### HACS (custom repository)

1. Open **HACS → Integrations**.
2. Open the three-dot menu and select **Custom repositories**.
3. Enter `https://github.com/acalcutt/oupes-mega-wifi-hass`.
4. Select **Integration**, click **Add**, and install **OUPES Mega WiFi**.
5. Restart Home Assistant.

### Manual

1. Copy `custom_components/oupes_mega_wifi/` into your HA config directory.
2. Restart Home Assistant.

## Quick Start

1. Add the **OUPES Mega WiFi** integration — configure ports.
2. Set up NAT rules on your firewall/router to redirect `47.252.10.9:8896` → HA.
3. Log in to discover devices.

See the [WiFi README](custom_components/oupes_mega_wifi/README.md) for the full
NAT rule table.

> **Optional:** onboarding a device that has never been on WiFi needs a
> Bluetooth handoff to send it credentials. Install the
> [BLE integration](https://github.com/acalcutt/oupes-mega-hass) alongside this
> one and the config flow will use it automatically; without it, the pairing
> steps are skipped and you provision the device with the Cleanergy app instead.

---

## WiFi or BLE?

| Scenario | Install |
|----------|---------|
| Device too far for BLE, or you want push telemetry | this integration |
| Simple local-only setup, device within BLE range | [oupes-mega-hass](https://github.com/acalcutt/oupes-mega-hass) |
| Want both channels for redundancy | both |

**WiFi requires network-level redirection.** The device firmware hardcodes the
cloud broker IP (`47.252.10.9`), so you need firewall NAT rules to intercept the
device's outbound connections and redirect them to your HA instance. This is
more powerful (push-based, real-time data, works at any distance) but involves a
more complex setup.

**BLE is the easiest path.** It works entirely over Bluetooth with zero network
configuration — just a USB Bluetooth adapter (or an ESPHome BLE proxy).

---

## Supported Models

| Series | Models |
|--------|--------|
| **Mega** | Mega 1, Mega 2, Mega 3, Mega 5 |
| **Exodus** | Exodus 1200, Exodus 1500, Exodus 2400, S024 Lite, S1 Lite |
| **Guardian** | Guardian 6000, HP2500, D5 V2 |
| **Other** | S2 V2, DC 800, LP350, LP700, PB300, UPS 1200, UPS 1800 |

Model-specific features (settings, entity names) are applied automatically
based on the product ID.

---

## Protocol Documentation

See [`debug_info/README.md`](debug_info/README.md) for the complete
reverse-engineered protocol reference, covering:

- WiFi TCP broker protocol, streaming activation sequence
- Telemetry attribute map (shared across BLE and WiFi)
- Cloud API endpoints (HTTP REST + SiBo)
- BLE GATT profile, pairing/claiming protocol, packet format
- Device firmware boot sequence

## Debug Tools

The [`debug_info/`](debug_info/) directory contains standalone tools:

| Script | Purpose |
|--------|---------|
| `scan_wifi_ports.py` | Scan device for open network ports |
| `pair_device.py` | BLE pairing + WiFi provisioning |
| `ble_confignet.py` | Send WiFi credentials to a paired device |
| `scan_ble.py` | Live BLE telemetry scanner |
| `parse_btsnoop.py` | Parse Android btsnoop HCI logs |
| `probe_key.py` | Test candidate device keys |
| `ble_diag.py` | BLE GATT diagnostics |
| `analyze_attr_csv.py` | Analyze BLE attribute debug logs |

---

## License

See [LICENSE](LICENSE).
