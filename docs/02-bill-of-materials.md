# 02 Bill of materials

Prices are approximate as of late 2026 and exclude shipping.

**No affiliate links anywhere in this repository.** Every link is a plain product link with tracking parameters stripped. If you fork this and add referral links, please say so in your README.

Links are provided for convenience and go stale quickly. **The spec column is what actually matters.** Match the specification, not the listing.

## Core parts

| Part | Spec | Approx. | Link | Notes |
|---|---|---|---|---|
| Peristaltic dosing pump | Kamoer NKP, 12V DC, 3mm ID x 5mm OD | $30 | [link](https://www.amazon.com/dp/B07GWJ78FN) | See selection notes below |
| Dry contact relay | Shelly 1 Gen4, UL listed | $28 | [link](https://www.amazon.com/dp/B0H8YW595V) | Must accept 12V DC power. See alternatives below |
| Silicone tubing | 3mm ID x 5mm OD, 9.8 ft | $9 | [link](https://www.amazon.com/dp/B0FMF45JZH) | Must match the pump's tube size |
| Panel mount barrel jack | 5.5 x 2.1 mm, 11 mm panel hole | $6 for 6 | [link](https://www.amazon.com/dp/B0GGN5MWH5) | Only one needed |
| 12V DC power supply | 12V 2A, centre positive, 5.5 x 2.1 mm plug | $13 for 2 | [link](https://www.amazon.com/dp/B077PW5JC3) | 1A is enough; headroom is free. Meter the polarity before use |
| Vinegar | 1 gallon, cleaning or distilled white | $4 | [link](https://www.walmart.com/ip/GV-Vinegar-128oz/15941853266) | Its own HDPE jug is the reservoir |
| Containment bucket | 2 gallon | $6 | [link](https://www.homedepot.com/p/The-Home-Depot-2-gal-Homer-Bucket-RG502HD/316355946) | Must exceed the jug's capacity |
| Water leak sensor | Any Home Assistant compatible | $25 | [link](https://www.amazon.com/dp/B07Z7QWJBP) | Goes in the bottom of the bucket. Reference build uses a YoLink YS7903, which needs a YoLink hub |
| Lever nut connectors, 3 conductor | WAGO 221-413, 24-12 AWG | $7 for 10 | [link](https://www.amazon.com/dp/B0GLHLRZR4) | The two junction points |
| Lever nut connectors, 2 conductor | WAGO 221-412, 24-12 AWG | $6 for 10 | [link](https://www.amazon.com/dp/B072PT3JNL) | Handy for the single O to pump run |
| Hookup wire | 18 AWG, a few feet | on hand | [link](https://www.amazon.com/dp/B0DXKKCS5R) | Any 18 AWG will do. The reference build used 18/3 thermostat cable left over from another job |
| Spare PVC cleanout cap | Matching your existing cap | $2 | hardware store | Drilled, so keep the original |
| M3 x 16 machine screws, nuts, washers | 2 of each | few $ | hardware store | Pump flange to lid |
| M3 x 10 machine screws | 4 | $2 | hardware store | Lid to box |

## Pump selection

The build was developed around a **Kamoer NKP 12V** with a 3mm ID x 5mm OD tube. Any similar 12V DC peristaltic dosing pump will work. What to match:

- **12V DC brushed motor.** Avoid stepper driven pumps, which need a driver board.
- **Tube size 3mm ID x 5mm OD.** If yours differs, the cap hole and tubing in the docs change accordingly.
- **Flow rate roughly 70 to 150 mL/min.** Faster pumps dose too coarsely for a one cup target. The measured unit in this build delivered about 135 mL/min.
- **Snap in pump head if available.** Tube replacement without tools is worth a lot when the pump lives in an attic.

The enclosure CAD assumes a **29 mm diameter, 48.5 mm long motor with a 54.5 x 40.3 mm flange and screws 47 to 48.5 mm apart.** The lid's mounting holes are slots, so the pitch tolerance is covered, but a very different motor needs the bore resized.

## Relay selection

The relay must satisfy three things:

1. **Dry contact output** so the pump is switched independently of the relay's own power
2. **12V DC input**, so one supply runs everything
3. **Local control** from Home Assistant

The Shelly 1 Gen4 meets all three. Note the important caveat about its Zigbee mode in [09 Troubleshooting](09-troubleshooting.md).

**Do not use the Shelly 1 Mini Gen4.** It has dry contacts but is AC powered only, with no DC input.

Other dry contact relay modules that take 12V DC work equally well. Generic Zigbee dry contact modules are around $17.

## Alternative: ESP32 controller

Instead of a commercial relay you can drive the pump from a microcontroller. This costs less and gives you a hardware enforced maximum runtime, at the price of soldering.

| Part | Approx. |
|---|---|
| Seeed XIAO ESP32C3 | $5 |
| IRLB8721PBF MOSFET | $1 |
| 100 ohm and 10k resistors | pennies |
| MP1584EN buck converter module | $2 |

Not yet built, so no links. These are stock parts available from any electronics supplier.

Full detail in [10 ESPHome](10-esphome.md).

**Use the IRLB8721, not the IRLZ44N** that most tutorials name. The IRLZ44N specifies its on resistance at 5V gate drive and is marginal on a 3.3V microcontroller pin.

## Printed parts

About 60 g of filament total. PLA is adequate for an attic in a temperate climate; consider PETG if yours runs very hot.

| File | Print time | Notes |
|---|---|---|
| `cad/Vinegar_Doser_Box_80x80x53` | ~2 h | Opening up, no supports |
| `cad/Vinegar_Doser_Lid_pump_mount` | ~40 min | Flat face down, boss up, no supports |
| `cad/DrainFlo_Badge_drop_A_full_tagline` | ~25 min | Optional, four colours |

Settings in [03 Enclosure](03-enclosure.md).

## Tools

- 3D printer, 0.4 mm nozzle
- Drill with a 3/16 inch bit for the cap and jug, and 11 mm or 7/16 inch for the barrel jack
- Calipers
- Multimeter, to confirm barrel jack polarity
- Measuring cup, for calibration
- Two small wrenches for the M3 hardware
- WAGO lever nuts or a small terminal block

## Not required

- No check valve. The peristaltic pump provides that.
- No flyback diode. At roughly 12 switching cycles a year, a 0.4 A motor will not measurably wear a 10 A relay.
- No fittings, barbs or adapters. The tubing runs unbroken.
