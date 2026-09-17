# 10 ESPHome controller

An alternative to the commercial relay. Costs less, requires soldering, and gives you something the Zigbee relay cannot: **a maximum runtime enforced on the microcontroller itself**, with no network involved.

If Home Assistant dies mid dose, the ESP32 still shuts the pump off.

## Which to choose

| | Commercial relay | ESPHome |
|---|---|---|
| Cost | ~$19 | ~$8 |
| Soldering | None | Yes |
| Hardware watchdog on Zigbee | **No** | **Yes** |
| Hardware watchdog on WiFi | Yes | Yes |
| Setup | Pair and go | Flash and go |

## Parts

| Part | Spec | Approx. |
|---|---|---|
| Seeed XIAO ESP32C3 | With the external antenna it ships with | $5 |
| IRLB8721PBF | Logic level N channel MOSFET, TO-220 | $1 |
| Resistors | One 100 ohm, one 10k | pennies |
| MP1584EN module | Adjustable buck converter | $2 |

**Use the C3, not the S3.** The C3 has no PCB antenna at all and relies entirely on the external one it ships with, which is what you want through attic framing. The S3 buys nothing here and runs warmer.

**Use the IRLB8721, not the IRLZ44N** that most tutorials name. The IRLZ44N specifies its on resistance at 5 V gate drive and is marginal on a 3.3 V GPIO pin. The IRLB8721 has guaranteed on resistance at lower gate voltages and much lower gate charge.

**No flyback diode needed.** At roughly 12 switching cycles a year against a 0.4 A motor, it earns nothing.

## Circuit

One 12 V supply enters through the panel barrel jack and feeds two things: the pump, and an MP1584EN buck converter that makes 5 V for the XIAO.

```
12V+ ──┬── Pump +
       └── MP1584EN IN+ ── (set to 5.0V) ── XIAO 5V pin

Pump − ── MOSFET drain
MOSFET source ── 12V−
MOSFET gate ── 100 ohm ── XIAO GPIO5
MOSFET gate ── 10k ── 12V−

12V− ── XIAO GND, MP1584EN IN−, MOSFET source   (all common)
```

**Set the buck to 5.0 V with a meter before connecting the XIAO.** These modules ship at an arbitrary voltage set by a multi turn trimpot. Feeding the 5 V pin anything above about 5.5 V destroys the board.

**The 10k pulldown is not optional.** Without it the gate floats during boot and the pump can twitch.

**All grounds must be common.** The gate voltage is measured against source, so an ungrounded reference means unpredictable switching.

### Two supplies instead

You can skip the buck and power the XIAO from a separate USB-C supply. It works, and removes the trimpot step. **You must still tie the two grounds together**, or the gate has no reference to the pump's return path.

The reason the buck is preferred: the enclosure has one hole for one power connector, and nowhere for a second cord.

## Pin choice

GPIO5, which is silkscreened D3. Confirm against the pinout for your board.

**Avoid GPIO2, GPIO8 and GPIO9.** They are strapping pins on the ESP32C3 and pulling them at boot changes how the chip starts.

## Two configurations

**[`esphome/drainflo.yaml`](../esphome/drainflo.yaml)** is the shareable project file. It contains no credentials, sets up Improv provisioning over both USB serial and Bluetooth, and carries the `dashboard_import` block that lets anyone adopt the device into their own ESPHome Device Builder. Flash this one.

**[`esphome/drainflo-personal.yaml`](../esphome/drainflo-personal.yaml)** is a conventional config using `!secret` for WiFi, API and OTA credentials. Use this if you would rather manage the device the normal way.

## Flashing

Flash the first time over USB at a bench, not in the attic. After that everything is OTA.

Provisioning is via Improv: either plug it into a computer and use the ESPHome web installer, or use Bluetooth from the Home Assistant app. No credentials are baked into the firmware.

## What it exposes

| Entity | Purpose |
|---|---|
| Dispense Dose | Runs one full dose |
| Prime Pump | 20 seconds, for filling the line |
| Stop Pump | Aborts everything and forces off |
| Dose Duration | Your calibrated seconds per cup |
| Pump Running | Binary sensor |
| Restart ESP | |

**The raw pump switch is deliberately not exposed.** It is marked internal, so everything goes through the dose script and the pump cannot be left running by a stray toggle in the Home Assistant UI.

## Two layers of runtime protection

**The script** turns the pump off after `dose_seconds`.

**The switch itself** kills it at `max_run_seconds`, 300 by default, through an `on_turn_on` automation with `mode: restart`. This runs on the ESP with no network involved, so a dead WiFi link or a crashed Home Assistant still ends with the pump off.

That second layer is the entire reason to choose this over the Zigbee relay. See [08 Safety](08-safety.md).

## Using it with the Home Assistant package

The package in `homeassistant/packages/drainflo.yaml` switches `switch.drainflo`. The ESPHome device does not expose a switch, it exposes a button.

Change the dose branch to press `button.drainflo_dispense_dose` instead of toggling a switch, and drop the delay and the turn off, since the device handles its own timing. The safety cutoff and startup force off branches become redundant but are harmless to leave in place.

**Containment is still required.** A hardware watchdog reduces the risk, it does not remove it.
