# 09 Troubleshooting

## Known issues

### Shelly 1 Gen4 ignores on_time in Zigbee mode

**Symptom.** You publish `{"state": "ON", "on_time": 300}` to the device's Zigbee2MQTT set topic. The relay turns on and never turns off. No error appears anywhere.

**Diagnosis.** Publish `{"state": "ON"}` to the same topic. If that works, your topic is correct and the `on_time` attribute specifically is being discarded by the device.

**Cause.** The Gen4 firmware in Zigbee profile does not implement the OnOff cluster's timed off behavior. Zigbee2MQTT exposes only a plain on/off state for it, with no countdown, auto off or `on_time` entity.

**Workarounds.**
- Accept it and rely on the software cutoffs in the package. This is what the reference build does.
- Run the relay on WiFi, where the Shelly web interface has a working auto off.
- Use the ESPHome controller, which enforces its own runtime limit.

### Shelly 1 Mini Gen4 will not work in this build

It has dry contacts, but it is **AC powered only**. The spec sheet lists 110 to 240 V~ with only L and N terminals. There is no DC input. The full size Gen4 is the one that takes 12V DC.

### Switching the Shelly to Zigbee resets its settings

Everything except the WiFi configuration is wiped when the firmware profile changes. Set any device side options **after** switching, not before.

Also: **Zigbee OTA updates are not supported.** Update the firmware while it is still in WiFi or Bluetooth mode, because you cannot do it once it is on Zigbee without switching back.

---

## Symptoms

### The relay pocket is too tight for the Shelly

The pocket is 42.05 mm and the relay is 42.0 mm. Printed parts commonly come in a couple of tenths undersize.

File the inner faces of the two rails. They are 1.6 mm ribs standing in open air with access from above, and they only locate the relay, so removing 0.2 mm from each face weakens nothing.

Alternatively set an X-Y contour compensation of about −0.1 mm and reprint.

### The barrel jack will not fit the hole

The hole is 11 mm, matching the panel hole these jacks specify. If yours is tight, run a 7/16 inch bit through it. PLA drills easily; go slow and back out to clear chips.

### The pump runs but nothing comes out

- **Not primed.** Run it longer on the first go.
- **Inlet pulling air.** The pickup tube may be above the liquid, or the jug is nearly empty.
- **No vent, or a blocked vent.** The jug cannot draw air back in, so it starves or collapses. Check the vent hole.
- **Running backwards.** Swap the two pump leads.

### The jug is collapsing

The vent hole is missing, too small, or plugged. 3 mm minimum.

### Vinegar siphons out of the jug on its own

The vent tube is reaching into the liquid instead of staying in the air space above it. Shorten it.

### Doses are coming out short

Either the pump was not primed when you calibrated it, or the pump tube has taken a set and flow has dropped. Re run [07 Calibration](07-calibration.md). If the number has moved a lot, replace the tube rather than just increasing the dose time.

### The dose counter is wrong

It is open loop. Home Assistant cannot see inside the jug. If you refilled without pressing Refilled, the count is low. Set `input_number.drainflo_doses_remaining` back to 16 manually.

### "Pump did not start" notification, but the relay looks fine

The automation commands the relay on, waits 5 seconds, then checks whether it reports on. A slow or congested Zigbee network can miss that window.

Check the relay's link quality. If it is consistently poor, either improve the mesh with a router closer to the attic, or lengthen the 5 second settle delay in the automation.

### The pump fault warning will not clear

By design. It does not self clear when the relay comes back, because a missed monthly dose stays missed until someone deals with it.

Dose manually if the line still needs it, then run `script.drainflo_clear_fault` or tap Dismiss on the notification.

### Nothing dosed on the 15th

Check, in order:

1. Was `switch.drainflo` available at 05:00? Look at its history.
2. Is `automation.drainflo_dosing` enabled?
3. Did you get a fault notification? Check `input_boolean.drainflo_pump_fault`.
4. Check the automation trace in Home Assistant, which will show which branch ran and where it stopped.
