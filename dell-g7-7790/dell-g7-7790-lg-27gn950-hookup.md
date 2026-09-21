# Dell G7 7790 → LG 27GN950-B Hookup Notes

Quick reference for connecting the Dell G7 7790 laptop to the LG 27GN950-B
27" 4K gaming monitor (Amazon ASIN B088D8JG3L).

## Size comparison

| Device            | Panel size |
|-------------------|------------|
| LG 27GN950-B      | 27"        |
| Dell G7 7790      | 17.3"      |

The external monitor is significantly larger — roughly 2.4× the screen area
of the laptop panel.

## Ports available

**Dell G7 7790 (video outputs):**
- HDMI 2.0
- Mini DisplayPort 1.4 (wired to the Nvidia GPU — RTX 2060/2070/2080 depending on config)
- Thunderbolt 3 (USB-C, also supports DisplayPort Alt Mode)

**LG 27GN950-B (video inputs):**
- 2× HDMI 2.0 (capped at 4K @ 60Hz)
- 1× DisplayPort 1.4 (full 4K @ 144Hz, native 3840×2160)

## Connection options

### Option A — HDMI (simplest)
- Works out of the box.
- Caps at **4K @ 60Hz** because both ends are HDMI 2.0.
- Fine for productivity; misses the monitor's 144Hz refresh rate.

### Option B — Mini DisplayPort → DisplayPort (recommended for 144Hz)
- Needed to unlock **4K @ 144Hz**.
- Two ways to wire it:
  1. Single **Mini DP → full-size DP cable**, rated DisplayPort 1.4
     (look for "VESA Certified DP8K" or "HBR3" labeling). Cleanest.
  2. **Mini DP → DP adapter** + the full-size DP cable included in the
     monitor box. Reuses the bundled cable but adds a connector.
- A DP 1.2 cable will cap at ~4K @ 60Hz — same as HDMI. Verify 1.4 rating.

### Option C — Thunderbolt 3 (USB-C) → DisplayPort
- Also viable via DisplayPort Alt Mode.
- Requires a USB-C → DP cable or adapter.

## What ships in the LG 27GN950-B box

- Full-size **DisplayPort 1.4** cable (DP-to-DP)
- HDMI cable
- USB 3.0 upstream cable
- Power brick

The included DP cable is full-size on both ends, so it does **not** plug
directly into the G7's Mini DP port. Either add an adapter or buy a
Mini DP → DP cable.

## TL;DR

- HDMI works today, but only 4K @ 60Hz.
- For the monitor's full 4K @ 144Hz, use the G7's Mini DisplayPort with a
  DP 1.4-rated Mini DP → DP cable (or adapter + bundled DP cable).
