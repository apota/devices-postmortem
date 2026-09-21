# Dell G7 7790 — Battery Replacement Notes

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

## After installing

1. Charge to 100% on AC, then run on battery to ~5% once to let Windows
   recalibrate the fuel gauge.
2. Re-run the battery report:
   ```powershell
   powercfg /batteryreport /output "$env:USERPROFILE\battery-report-new.html"
   ```
3. Confirm the new "Design Capacity" reads ~60,000 mWh and "Full Charge
   Capacity" is close to it.
