# 11 Shelly device setup

Everything done on the relay itself, in the order it should be done. These settings live in the device's own config, not in Home Assistant, and they are what protect you when Home Assistant is not there.

Written against a Shelly 1 Gen4 on firmware 2.0.0. Earlier firmware puts some of these in different places, noted where it matters.

## Decide the protocol first

The Gen4 runs WiFi, Zigbee, Matter and Bluetooth. **It will run WiFi and Zigbee at the same time**, which turns out to be the best configuration for this build.

| | WiFi | Zigbee |
|---|---|---|
| Local control | Yes | Yes |
| Device side auto off | **Yes** | **No** |
| Availability tracking in HA | **Yes** | Depends on your Z2M settings |
| Device web interface | Yes | No |
| Firmware updates | Yes | No |

**The auto off timer is the deciding factor.** See [09 Troubleshooting](09-troubleshooting.md) for the full finding, but in short: Zigbee mode silently ignores the timer, so if you only run Zigbee, nothing on the device will ever stop a stuck pump.

Running both gives you Zigbee control plus the WiFi web interface and proper availability reporting. That is what the reference build does.

## 1. First power up and WiFi

Power the relay. It broadcasts an access point named after the device.

Join that network and browse to **http://192.168.33.1**.

Go to **Settings > WiFi**, enter your network and password, and apply. The web interface shows the new IP once it connects. From then on you reach it at that address.

**Give it a DHCP reservation** in your router. The Home Assistant integration pins to the IP and the web interface is easier to find when the address holds.

## 2. Update the firmware

Do this before anything else, because **Zigbee mode cannot do OTA updates.** If you switch to Zigbee first you have to switch back to update.

Either from the device's own Settings, or from the Home Assistant update entity once the Shelly integration has it. Both trigger the same process.

Do not power cycle during the update.

## 3. Turn off Cloud

**Settings > Cloud**, toggle Enable off.

By default the device maintains a connection to a Shelly server so their app can reach it remotely. Nothing in this build uses that. Turning it off means the relay only answers to your own network.

You lose remote access from the Shelly app. Local app control and Home Assistant are unaffected.

## 4. The three settings that matter

These are the device side protections. All three survive firmware updates, but verify them afterwards anyway.

### Auto OFF: 300 seconds

**On firmware 2.0.0:** Home page, click the **output**, then the **Timers** tab under Automations. On 2.0.0 this may also appear on its own settings page.

**On 1.7.x:** same place, but the Enable checkbox does not exist. A non zero value is the enable.

Set **Auto OFF** to **300** and leave **Auto ON** at 0, which disables it.

This is the hardware watchdog. The relay counts down on its own, so a dead network, a crashed Home Assistant or a hung automation all still end with the pump off. Set it comfortably above your dose time. A 110 second dose against a 300 second ceiling leaves plenty of margin while still catching a stuck relay well before a jug empties.

### Output type: Detached

Home page, click the output, **Input/Output settings**, set output type to **Detached**.

The default is Toggle, which ties the relay to the physical SW input. **With nothing wired to SW that input floats**, and a floating input can read as a state change. Detached separates it entirely, so nothing physical can close the relay.

### Action on power on: Turn OFF

Same page, below the output type.

**Do not use "Restore last known state."** That is the right default for a light and the wrong one for a pump. If power blips mid dose, restore brings the relay back closed, and the auto off timer does not resume because the device just booted with no memory of a countdown. You would have a pump running with nothing counting down.

Turn OFF means a power interruption always ends with the pump stopped. The worst case is a missed dose, which gets caught next month.

**Current state of the switch** is worse still on a Detached setup with a floating input, since the relay would follow an unconnected pin on every boot.

## 5. Optional: add Zigbee alongside

If you want Zigbee control as well, do it now, after the firmware update and after the settings above.

**Switching wipes every setting except the WiFi configuration.** Auto off, output type and power on action all reset, so redo section 4 afterward and verify.

Two ways to switch:

- **Web interface.** A Matter to Zigbee toggle in Settings, then a Zigbee menu with Start pairing. Status reads Steering while discoverable and Joined on success.
- **Button.** Five quick presses switches profile and opens a 3 minute inclusion window. Three presses reopens it. The presses have to be genuinely fast.

## 6. Verify

After everything, confirm all three:

- Auto OFF reads 300 and is enabled
- Output type is Detached
- Action on power on is Turn OFF

Then **test the auto off.** Turn the relay on and walk away. It should shut off by itself at 300 seconds with Home Assistant doing nothing.

Do this with **water, not vinegar.** Five minutes at 135 mL/min is about 680 mL down the drain for a test. Put the inlet in a container of water and the outlet in a bucket.

If you would rather not wait five minutes, set Auto OFF to 30, test, then put it back to 300.

**Test it once.** It is the layer that matters most and the one you would least want to discover was never working.

## What you get in Home Assistant

Adding the device to the Shelly integration over WiFi gives you, alongside the switch:

- **Device temperature.** The reference build read 99.9F inside the enclosure at mid morning in September. Useful, given most small pumps are rated only to 40C.
- **Uptime**, which is how you spot unexpected reboots
- **Signal strength**
- **Firmware update entity**
- **Proper availability**, which is the thing Zigbee could not provide

If the device was discovered and ignored at some earlier point, it will be sitting under **Settings > Devices & Services > Ignored**. Un-ignore it rather than hunting for a way to add it manually.
