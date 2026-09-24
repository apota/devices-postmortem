# Dell G7 7790 — KYY 15.6" Portable Monitor Compatibility

Whether the [KYY 15.6" FHD portable monitor (Amazon B088D8JG3L)](https://www.amazon.com/dp/B088D8JG3L)
can connect to the Dell G7 7790, and which of the included cables to use.

---

## 1. Quick answer up front

**Yes — the KYY monitor connects to the Dell G7 7790 three different ways.**
The best option is **USB-C to USB-C** (single cable, video + power from the
laptop's Thunderbolt 3 port). Mini-HDMI to HDMI works too and frees the
USB-C port, but requires the KYY's separate power brick.

---

## 2. Relevant ports on the Dell G7 7790

Per Dell's spec sheet for the G7 7790 (17.3" gaming laptop, 2019):

| Laptop port | Spec | Drives an external display? |
|---|---|---|
| **USB-C / Thunderbolt 3** (left side) | TB3, DisplayPort 1.2 Alt Mode, USB 3.1 Gen 2, 15 W PD out | **Yes** — video + limited power out |
| **HDMI 2.0b** (rear) | Full-size HDMI, up to 4K @ 60 Hz | **Yes** |
| **Mini-DisplayPort 1.4** (rear) | mDP 1.4 | Yes — but KYY has no DP input |
| USB-A 3.1 ×3 | Data only | No |

The G7 7790's USB-C port is a **full Thunderbolt 3** port with DisplayPort
Alt Mode, which is exactly what the KYY listing calls out ("as long as your
device supports Thunderbolt 3 or 3.1 USB-Type-C"). So the "1 cable solution"
path is available.

---

## 3. The three ways to connect

### Option A — USB-C to USB-C (recommended)

- **Cable:** the USB-C ↔ USB-C cable included in the KYY box.
- **Laptop port:** Thunderbolt 3 port on the left side of the G7 7790.
- **Monitor port:** either of the two Full-Function USB-C ports on the KYY.
- **Power:** the G7 7790's USB-C port supplies ~15 W, which is enough to
  drive the KYY's screen at normal brightness. No separate power brick
  needed for typical use.
- **Caveat:** at maximum brightness or high volume the KYY can draw more
  than 15 W. If the screen dims or flickers, plug the KYY's power brick
  into its **second** USB-C port and keep the video cable on the first.
  This splits power (from brick) and video (from laptop) across the two
  USB-C ports.

### Option B — Mini-HDMI to HDMI

- **Cable:** the Mini-HDMI (monitor side) ↔ full-size HDMI (laptop side)
  cable included in the KYY box.
- **Laptop port:** HDMI 2.0b port on the rear of the G7 7790.
- **Power:** HDMI carries no power, so the KYY **must** be powered from
  its included USB-C power brick (into either USB-C port on the monitor).
- **When to use this:** if the Thunderbolt 3 port is already occupied
  (external SSD, dock, charger), or if you want to preserve full USB-C
  bandwidth for something else.

### Option C — USB-C video + external USB-C power

- Same as Option A but with the KYY's power brick plugged into the
  monitor's second USB-C port.
- Use this if Option A dims/flickers under load, or if you want to keep
  the laptop battery from being drained by the monitor.

---

## 4. Resolution and refresh rate

The KYY is 1920×1080 @ 60 Hz. Both the G7 7790's Thunderbolt 3 port
(DisplayPort 1.2 Alt Mode) and its HDMI 2.0b port can drive this
comfortably — 1080p60 is well below the bandwidth ceiling of either.
No scaling or refresh-rate compromises.

---

## 5. What's in the KYY box (cables)

Based on the standard KYY 15.6" bundle for this SKU:

1. USB-C to USB-C cable (video + power, single-cable mode)
2. Mini-HDMI to HDMI cable
3. USB-C to USB-A cable + wall power adapter (power-only, for devices
   whose USB-C can't supply enough wattage)
4. Smart-cover/stand (not a cable, but included)

Verify against the "What's in the Box" section on the Amazon listing at
purchase time — KYY has revised the bundle a few times.

---

## 6. Known gotchas

- **USB-C port confusion.** The G7 7790 has only one USB-C port and it is
  Thunderbolt 3. Some earlier Dell G-series laptops shipped with USB-C
  that was data-only (no DisplayPort Alt Mode). The 7790 is fine, but if
  you also own an older G3/G5, don't assume the same cable will work
  there.
- **BIOS-level display.** The KYY over USB-C will not show BIOS/POST
  screens on some Dell laptops until Windows loads the graphics driver.
  If you need to see BIOS output on the KYY, use the HDMI connection
  instead — HDMI is picked up at firmware level.
- **Audio.** The KYY has built-in speakers and a 3.5 mm out. Over USB-C
  or HDMI, audio is carried on the same cable — select "KYY" (or the
  generic USB audio device name) in Windows sound output.
