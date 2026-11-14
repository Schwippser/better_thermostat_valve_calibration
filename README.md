# Thermostat Calibration Watchdog

<a href="https://www.buymeacoffee.com/tomreim0s" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-blue.png" alt="Buy Me a Coffee" style="height: 60px !important;width: 217px !important;" ></a>

🇬🇧 English | [🇩🇪 Deutsch](README.de.md)

A Home Assistant blueprint that automatically starts a **valve recalibration** of thermostatic radiator valves (TRVs) when they have lost their calibration.

[![Open your Home Assistant instance and show the blueprint import dialog.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/Schwippser/better_thermostat_valve_calibration/blob/main/thermostat_calibration_watchdog.yaml)

> **Note:** This blueprint only works with TRVs that support valve calibration and let you start it from Home Assistant through a button.

## Why this blueprint?

Some TRVs, for example the **Sonoff TRVZB**, can lose their valve calibration, often because of the heating system itself. The valve then reports a position of 0 % although it is physically open, so the room keeps heating above the target temperature. The reported valve position is therefore **not usable** for detecting the problem.

This blueprint detects the symptom instead: the room stays clearly above the target temperature for a long time and does not cool down. In that case it presses the calibration button of the thermostat(s) in that room.

## How it works

1. **Trigger:** the actual temperature is at least *X* K above the target temperature (default 2.0 K) **without interruption** for a given time (default 1 hour). The climate entity must not be `off`.
2. **Trend check:** the automation remembers the actual temperature, waits (default 30 minutes) and checks again.
3. **Decision:** the calibration is only started if the room is **still** at least *X* K too warm **and** the temperature has not dropped by more than the tolerance (default 0.2 K). This filters out the normal slow cooling after you lowered the target temperature.
4. **Action:** an optional push notification is sent and all selected calibration buttons are pressed.

After a calibration the automation only triggers again once the difference has dropped below the threshold at least once and then exceeded it again. This prevents endless calibration loops.

## Requirements

- Home Assistant **2024.10.0** or newer
- A `climate` entity per room providing the attributes `current_temperature` and `temperature`
- One or more `button` entities that start the valve calibration of the TRVs (**the TRV must support valve calibration** and expose a `button` entity for it; without that, this blueprint cannot be used)
- Optional: the Home Assistant Companion app for push notifications

Developed and tested with **Sonoff TRVZB** thermostats (Zigbee2MQTT) combined with **Better Thermostat**. Better Thermostat can group several TRVs into one climate entity, which is why the blueprint accepts several calibration buttons. Other combinations should work as long as the requirements above are met, but have not been tested by the author.

## Installation

**Import button:** click the badge at the top of this page.

**Manual:**
1. Copy `blueprints/automation/thermostat_calibration_watchdog.yaml` to `/config/blueprints/automation/<your_folder>/` on your Home Assistant.
2. If the blueprint does not show up in the list, reload automations (Developer tools → YAML → *Reload automations*) or restart Home Assistant.

## Usage

Create **one automation per room**: Settings → Automations & scenes → Create automation → Use blueprint → *Thermostat Calibration Watchdog*.

### Settings

| Input | Description | Default |
|---|---|---|
| **Climate entity (room)** | Climate entity providing actual and target temperature (e.g. the Better Thermostat entity of the room). | – |
| **Calibration button(s)** | One or more buttons that start the valve calibration. All are pressed. | – |
| **Temperature difference** | Minimum difference *actual − target* in Kelvin. | 2.0 K |
| **Minimum duration above threshold** | The difference must exist this long without interruption. | 1 h |
| **Trend check delay** | Waiting time before the second check. | 30 min |
| **Trend tolerance** | Allowed temperature drop during the delay that still counts as "not falling". | 0.2 K |
| **Mobile app device** | Companion app device for the push message. Empty = no notification. | empty |
| **Notification title / message** | Texts of the push message. The message may contain templates, the variable `climate_entity` is available. | English text |

### Example (room with two TRVs)

- Climate entity: `climate.heating_bathroom`
- Calibration buttons: `button.thermostat_bath_1_valve_travel_calibration`, `button.thermostat_bath_2_valve_travel_calibration`

## Choosing the threshold

| Variant | Difference | Duration | Use case |
|---|---|---|---|
| Sensitive | 1.5 K | 45 min | Fast reaction, more false alarms possible |
| **Default** | **2.0 K** | **60 min** | Good balance |
| Conservative | 3.0 K | 90 min | Rooms with strong sun or other heat sources |

A normal overshoot of a TRV is about 0.5 to 1 K. Direct sun, a fireplace or an oven can keep a room above the target for a long time and may cause an unnecessary calibration. This is harmless but can be reduced with a higher threshold.

## Notes

- The automation runs in `single` mode with silent `max_exceeded`: while one run is waiting for the trend check, further triggers of the same room are ignored.
- The calibration itself is performed by the thermostats and can take several minutes, during which the valves move.
- If the room stays too warm after a calibration, the cause is probably not the valve calibration.

## Disclaimer

Use at your own risk. The author takes no responsibility for damage or malfunction of heating systems or devices.

## License

This project is licensed under the [MIT License](LICENSE).
