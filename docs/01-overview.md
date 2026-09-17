# 01 Overview

## The problem

An air handler pulls moisture out of the air and drains it away through a PVC condensate line. That line is dark, permanently damp and full of organic debris, which is ideal for algae and biofilm. Left alone it clogs.

When the primary line clogs, water backs up into the drain pan and out the secondary line, which usually discharges somewhere deliberately annoying so you notice. If the secondary is also blocked, or there is no float switch, the pan overflows into the ceiling.

The standard prevention is a cup of distilled white vinegar down the cleanout once a month. It works. The problem is remembering to do it, and in many houses the cleanout is in an attic.

## The approach

A small peristaltic pump, a jug of vinegar and a relay that Home Assistant can switch. Once a month the automation closes the relay for a measured number of seconds, the pump delivers one cup, and the relay opens again.

Three properties drove the design:

**Local only.** No cloud account, no vendor API, nothing that stops working when a company changes its business model. The relay speaks Zigbee or WiFi to your own network.

**No fittings in the liquid path.** The pump's own silicone tubing runs unbroken from the jug to the drain line, pressed through a hole drilled in a spare cleanout cap. Every fitting is a potential leak, so there are none.

**Containment by default.** The jug sits inside a bucket sized to hold more than the jug contains. This is not optional.

## How a dose works

```
05:00 on the 15th
        |
        v
Is the relay reachable?  ---- no ---->  raise fault, notify, stop
        |
       yes
        v
Close the relay
        |
        v
Wait 5 seconds, did it actually report on?  ---- no ---->  force off, raise fault, notify
        |
       yes
        v
Wait the remaining dose time
        |
        v
Open the relay
        |
        v
Decrement dose counter, update next due date, notify success
```

The verification step matters. A relay that has silently dropped off the network will accept a turn on command that goes nowhere, and without checking you would believe the drain line was dosed when it was not.

## Components of the system

**Pump.** A 12V DC peristaltic dosing pump. Peristaltic because the rollers occlude the tube when idle, which means the pump itself blocks backflow and no check valve is needed. Also self priming and tolerant of running dry.

**Relay.** A dry contact relay that can be powered from the same 12V supply. Dry contact rather than a mains switch means one power supply for the whole build.

**Controller.** Home Assistant. It holds the schedule, runs the dose, verifies it, counts remaining vinegar and raises the alarms.

**Enclosure.** A printed box that holds the relay and carries the pump on its lid, so the pump motor hangs inside and the wet end stays outside.

**Containment.** A bucket under the jug, with a leak sensor in the bottom of it.

## Design decisions worth explaining

**Dose by time, not by volume.** There is no flow meter. The pump runs at a known rate, so the dose is a calibrated number of seconds. This drifts as the pump tube takes a set, which is why [07 Calibration](07-calibration.md) tells you to re measure occasionally.

**05:00 on a fixed day.** Early morning puts the attic near its daily minimum temperature, which matters because these pumps are typically rated only to 40C and an attic exceeds that by afternoon. A fixed day of the month rather than a rolling 30 day countdown keeps the schedule predictable and lets you avoid whatever day your other notifications land on.

**The counter is open loop.** Home Assistant cannot see inside the jug. The dose counter is a budget that decrements on each dose and resets when you tell it you refilled. If you refill and never acknowledge it, the count drifts low. The consequence is a false low vinegar warning, not a failure, but it is worth knowing.

**The fault flag does not self clear.** If a dose is missed, the warning stays up until you dismiss it. A relay coming back online does not mean the drain line got its vinegar.

## Next

Start with [02 Bill of materials](02-bill-of-materials.md).
