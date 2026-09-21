# Dell G7 7790 — Battery Replacement Notes

## Contents

- [Why replace](#why-replace)
- [Battery specs (Dell G7 7790, 17.3")](#battery-specs-dell-g7-7790-173)
- [How to verify before buying](#how-to-verify-before-buying)
- [Where to buy](#where-to-buy)
  - [What to look for in a listing](#what-to-look-for-in-a-listing)
- [Compatibility checks on specific listings](#compatibility-checks-on-specific-listings)
  - [Amazon B08XVX3S24 — 1F22N (Alienware M15/M17, G7 15 7590) — NOT COMPATIBLE](#amazon-b08xvx3s24--1f22n-alienware-m15m17-g7-15-7590--not-compatible)
- [After installing](#after-installing)

## Why replace

Battery report from the failing G7 7790 (see screenshot in postmortem):

| Metric                        | Value       |
|-------------------------------|-------------|
| Design capacity               | 60,010 mWh  |
| Current full-charge capacity  | 9,166 mWh   |
| Retained capacity             | ~15.3%      |
| Health loss                   | ~85%        |

Charger and charging circuit are fine — battery went from 49% → 54% on AC
during the observation window. The cell itself is what needs replacing.

## Battery specs (Dell G7 7790, 17.3")

The G7 17 7790 ships with a **60 Wh, 4-cell Li-ion** battery. Dell part
numbers that fit this chassis:

| Dell P/N   | Compatible P/Ns              | Capacity | Voltage |
|------------|------------------------------|----------|---------|
| **MC34Y**  | 33YDH, PVHT1, 99NF2, 0MC34Y  | 60 Wh    | 15.2 V  |

The 60 Wh / 15.2 V / 4-cell configuration matches the 60,010 mWh design
capacity in the battery report exactly, so **MC34Y** (or any of the
cross-referenced part numbers above) is the correct replacement.

> Note: the same G7 chassis was also sold with a 90 Wh 6-cell option
> (Dell P/N **W7NKD** / 8B/3H, higher-end SKUs). Confirm which one is
> currently installed by opening the bottom cover and reading the sticker
> before ordering — do not rely on the model number alone.

## How to verify before buying

1. Power off, unplug AC, remove the bottom cover (Phillips #0, ~10 screws).
2. Read the sticker on the battery. Look for one of: **MC34Y**, 33YDH,
   PVHT1, 99NF2 (60 Wh) or **W7NKD** (90 Wh).
3. Match the part number on the replacement listing to what's installed.

## Where to buy

I'm not going to hardcode specific Amazon product URLs here — third-party
battery listings churn constantly and dead links are worse than no links.
Use these search URLs instead; they'll always return the current listings:

- Amazon search — MC34Y:
  https://www.amazon.com/s?k=Dell+G7+7790+MC34Y+battery
- Amazon search — 33YDH (cross-reference):
  https://www.amazon.com/s?k=Dell+G7+17+7790+33YDH+battery+60Wh
- Amazon search — PVHT1 (cross-reference):
  https://www.amazon.com/s?k=Dell+PVHT1+battery
- Dell official parts (most reliable, usually pricier):
  https://www.dell.com/support/home/en-us/product-support/product/g-series-17-7790-laptop/drivers
  → Parts & Accessories tab, search "MC34Y".

### What to look for in a listing

- Part number **MC34Y** (or 33YDH / PVHT1 / 99NF2) printed in the title
  or bullet points.
- **60 Wh, 15.2 V, 4-cell** — reject anything that says 11.4 V, 3-cell,
  or a different Wh rating.
- Compatibility line explicitly mentions **G7 17 7790** (not just "G7"
  or "G3/G5" — those use different batteries).
- Prefer sellers offering a **12-month warranty**; aftermarket cells vary
  wildly in real-world capacity.

## Compatibility checks on specific listings

### Amazon B08XVX3S24 — 1F22N (Alienware M15/M17, G7 15 7590) — NOT COMPATIBLE

Listing: https://www.amazon.com/1F22N-Replacement-Compatible-Alienware-5590-D2783W/dp/B08XVX3S24

**Verdict: do not buy this for the G7 17 7790.** The 7590 in the title is
a different laptop — Dell G7 **15** 7590, a 15.6" chassis — not the
17.3" **7790** we need.

| Spec           | 1F22N (this listing)              | MC34Y (what the 7790 needs) |
|----------------|-----------------------------------|-----------------------------|
| Capacity       | 76 Wh                             | 60 Wh                       |
| Voltage        | 11.4 V                            | 15.2 V                      |
| Cell count     | 6-cell                            | 4-cell                      |
| Fits chassis   | Alienware M15/M17, G7 15 7590, G5 15 5590 | G7 17 7790          |

The voltage mismatch alone (11.4 V vs 15.2 V) means the charging circuit
would not regulate correctly even if the connector physically mated, and
the 6-cell 1F22N is a different physical size than the 4-cell bay in the
17.3" chassis.

Rule of thumb: **any Dell replacement battery listed as compatible with
"7590" but not "7790" is the wrong one** — the digit swap is a different
laptop generation and form factor.

## After installing

1. Charge to 100% on AC, then run on battery to ~5% once to let Windows
   recalibrate the fuel gauge.
2. Re-run the battery report:
   ```powershell
   powercfg /batteryreport /output "$env:USERPROFILE\battery-report-new.html"
   ```
3. Confirm the new "Design Capacity" reads ~60,000 mWh and "Full Charge
   Capacity" is close to it.
