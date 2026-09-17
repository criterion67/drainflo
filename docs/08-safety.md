# 08 Safety

Read this before building. It is the most important document in the repository.

## What you are building

A pump that moves liquid, mounted above a ceiling, with software deciding when it stops.

That is the whole risk in one sentence. Everything below follows from it.

## The primary failure mode

**The relay closes and does not open.**

If that happens and nothing intervenes, the pump keeps running until the jug is empty. A gallon of vinegar goes into your drain line far faster than it can drain, backs up, and overflows the pan into the ceiling.

## What the Shelly does not do

**In Zigbee mode the Shelly 1 Gen4 provides no device side auto off.**

The Zigbee `on_time` attribute is silently ignored. Publishing `{"state": "ON", "on_time": 300}` turns the relay on and it stays on indefinitely, with no error and no indication that the timer was dropped. A plain `{"state": "ON"}` to the same topic works, which confirms the topic is correct and the attribute specifically is being discarded.

This is not documented by Shelly. It was found by testing, and it is the reason this document exists.

**The consequence: the off command depends entirely on Home Assistant.** If Home Assistant is down, restarting, or has lost the network when a dose is in progress, nothing in the relay will stop the pump.

## What protects you

Four layers, in order from first to last.

**1. The dose sequence turns the pump off.** Normal operation. Turn on, wait, turn off.

**2. A safety cutoff automation.** Triggers when the switch has been on for 6 minutes against a 110 second dose, forces it off, and pushes an urgent notification. This catches a dose sequence that died part way through.

**3. A startup force off.** On every Home Assistant start, the switch is commanded off. This catches the case where Home Assistant crashed mid dose and came back with the relay still closed.

**4. Physical containment.** The jug sits in a bucket larger than the jug. Even if every software layer fails, the vinegar that is not already in the drain line ends up in the bucket.

**The gap that remains:** Home Assistant down for an extended period with the relay closed. Layers 1 to 3 are all software. Layer 4 is what stands between that scenario and your ceiling.

## Requirements, not suggestions

- **A containment vessel that holds more than the jug does.** A 1 gallon jug goes in a 2 gallon bucket.
- **A leak sensor in the bottom of that vessel.** Containment without detection just delays the discovery.
- **Do not skip the safety cutoff automation.** It is in the package for a reason.

## If you want a real hardware watchdog

Two options:

**Run the relay on WiFi instead of Zigbee.** The Shelly's web interface exposes a genuine auto off setting that the device enforces itself. Set it to 300 seconds. Verify it actually works by watching a test dose before you trust it.

**Use the ESPHome controller instead.** [esphome/drainflo.yaml](../esphome/drainflo.yaml) enforces a hard maximum runtime on the microcontroller itself. A dead network or a crashed Home Assistant does not affect it.

## Chemical notes

**Use distilled white vinegar, 5 percent acetic acid.** Not cleaning vinegar at higher concentration, not bleach.

**Never use bleach in a condensate drain line.** It corrodes the evaporator coil and the drain pan.

**If you reuse a bleach jug, rinse it thoroughly and let it dry open first.** Residual hypochlorite meeting acetic acid releases chlorine gas, and an attic is a poor place for that.

**Silicone tubing is fine with dilute acetic acid.** It is rated excellent for 10 percent acetic acid in water. Metal fittings are where the compatibility questions start, which is another reason this build has none.

## Temperature

Most small peristaltic pumps are rated for 0 to 40C ambient. An attic exceeds that on a summer afternoon in most of the United States.

Dosing in the early morning puts the run near the attic's daily minimum, which is why the schedule is at 05:00. Idle storage at high temperature is much less stressful than running at it, but it is a real constraint and worth knowing if you are considering a different schedule.

## Electrical

Everything in this build is 12V DC. There is no mains wiring anywhere.

If you substitute a mains switched relay, that is a different project with different requirements, and this document does not cover it.
