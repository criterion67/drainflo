# 07 Calibration

The dose is a number of seconds, not a measured volume. You have to find out how many seconds your pump takes to deliver one cup.

**Do this on a bench with water, over a sink. Not in the attic with vinegar.**

## Setup

Inlet tube into a bowl of water, outlet tube into a container. Fill the bowl generously, at least a quart, so the pump cannot suck it dry and start pulling air part way through the run.

## Prime first

Run the pump until water flows steadily out of the outlet, then stop and empty the container.

If you skip this, the priming time gets baked into your number and every real dose comes out short.

## Measure

Two methods. The second is easier to do accurately.

**Method A.** Mark one cup on your collection container first: measure a cup of water, pour it in, mark the line, empty it. Then run the pump and time how long it takes to reach the line.

**Method B.** Run for exactly 60 seconds, then pour what you collected into a measuring cup and read it. Scale from there. If you got 80 mL in a minute, a cup takes about 178 seconds.

One US cup is 237 mL.

## Set the dose time

Add about 5 seconds of margin to whatever you measured. Slightly overdosing is harmless; underdosing defeats the purpose.

The reference build measured **105 seconds per cup**, roughly 135 mL/min, and uses a **110 second** dose.

Edit the two delays in `automation.drainflo_dosing`. The sequence is a 5 second settle after turn on, then the remainder. For a 110 second total the second delay is 105 seconds. If your dose time is different, change that second delay.

## Re measure occasionally

Peristaltic tubing takes a set where the rollers compress it, and flow drops over time. Re run this test once a year.

**If a cup starts taking noticeably longer, that is your signal to replace the pump tube**, not just to increase the dose time.

## Vinegar versus water

Close enough. At 5 percent acetic acid the viscosity is near identical to water, so the number transfers.
