# 06 Home Assistant

## Pairing the relay

### Zigbee (Zigbee2MQTT)

The Shelly 1 Gen4 ships in Matter profile and must be switched to Zigbee first.

**Update the firmware before switching.** Zigbee OTA is not supported, so once it is on Zigbee you cannot update it without switching back.

Two ways to switch:

- **Web interface.** Settings has a Matter to Zigbee toggle, then a Zigbee menu with Start pairing. Status reads Steering while discoverable and Joined on success.
- **Button.** Five quick presses switches profile and opens a 3 minute inclusion window. Three presses reopens it. The presses have to be genuinely fast.

**Switching wipes every setting except the WiFi configuration.** Configure anything device side after the switch, not before.

Name the device `DrainFlo` in Zigbee2MQTT, which yields `switch.drainflo`. Resist adding a room or descriptor, since Z2M builds the entity_id from the friendly name.

### WiFi

Power the relay, join the access point it broadcasts, and browse to `http://192.168.33.1`. Point it at your network, then give it a DHCP reservation.

**On WiFi you get a real auto off timer** in the Shelly web interface. Set it to 300 seconds. This is a genuine device side watchdog and it is the main reason to prefer WiFi for this build. Full device side procedure in [11 Shelly setup](11-shelly-setup.md); see also [08 Safety](08-safety.md).

## Installing the package

Enable packages in `configuration.yaml` if you have not already:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

Copy [`homeassistant/packages/drainflo.yaml`](../homeassistant/packages/drainflo.yaml) into `config/packages/` and restart Home Assistant.

### Three things to change first

| Placeholder | Occurrences | Change to |
|---|---|---|
| `notify.mobile_app_pixel_9` | 9 | Your own notify target |
| `switch.drainflo` | 11 | Your relay's entity_id |
| `delay: seconds: 105` | 1 | Your calibrated dose time minus 5 |

The `Quiet Notices` channel is Android specific. On first use Android creates it silent at low importance. **iOS users should remove the `channel` and `importance` keys** from the notification data blocks.

### Seed the helper values

After the restart:

- `input_number.drainflo_doses_remaining` to **16** for a full gallon
- `input_datetime.drainflo_next_dose` to **the 15th of next month**

## What gets created

**Helpers**

| Entity | Purpose |
|---|---|
| `input_number.drainflo_doses_remaining` | Open loop dose budget, 16 per gallon |
| `input_boolean.drainflo_pump_fault` | Drives the fault card, does not self clear |
| `input_boolean.drainflo_refill_active` | Drives the refill card |
| `input_datetime.drainflo_refill_snooze_until` | 7 day snooze deadline |
| `input_datetime.drainflo_next_dose` | Display only readout |

**Scripts**

| Entity | Purpose |
|---|---|
| `script.drainflo_dose_complete` | Updates the next dose readout, logs |
| `script.drainflo_refilled` | Resets the budget to 16 |
| `script.drainflo_refill_snooze` | Pushes the snooze out 7 days |
| `script.drainflo_clear_fault` | Dismisses the fault state |

**Automations**

| Entity | Purpose |
|---|---|
| `automation.drainflo_dosing` | Dose, verify, safety cutoff, startup force off, offline warning |
| `automation.drainflo_refill_reminder` | Daily low vinegar level check |

## Testing before you trust it

**Run `automation.drainflo_dosing` manually.** Triggering by hand skips the day of month condition, so it doses regardless of the date. Do this with the outlet tube in a container, not in the drain line.

Watch for: the relay closing, the pump running for your dose time, the relay opening on its own, a success notification, and the counter dropping by one.

Afterwards, reset the counter and the next dose date if you want a clean slate.

**Test the safety cutoff too.** Turn `switch.drainflo` on manually and leave it. After 6 minutes it should force off and push on the urgent channel. This is the layer that matters most, so confirm it works rather than assuming.

## Dashboard cards

Two conditional cards, one per boolean, in whatever notification area your dashboard uses:

```yaml
type: conditional
conditions:
  - condition: state
    entity: input_boolean.drainflo_pump_fault
    state: "on"
card:
  type: markdown
  content: >-
    ### DrainFlo pump fault

    The monthly dose did not complete. Dose manually if the line needs it,
    then dismiss.
  card_mod: {}
```

```yaml
type: conditional
conditions:
  - condition: state
    entity: input_boolean.drainflo_refill_active
    state: "on"
card:
  type: markdown
  content: >-
    ### DrainFlo vinegar low

    {{ states('input_number.drainflo_doses_remaining') | int }} doses left.
```

Add tap actions calling `script.drainflo_clear_fault` and `script.drainflo_refilled` respectively. Put a confirmation on both if the dashboard lives on a wall tablet where a stray tap is plausible.

## A note on the reference instance

The reference build reuses a helper named `input_datetime.hvac_drain_line_due_date` and a script named `script.hvac_drain_line_cleaned`, inherited from the manual reminder system DrainFlo replaced. The package uses clean `drainflo_` names throughout. Functionally identical.
