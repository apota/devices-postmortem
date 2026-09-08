# Dell Latitude 7410 — Diagnostic Steps

## Table of contents

- [Conclusion / Findings](#conclusion--findings)
- [What we already know from the BIOS screenshots](#what-we-already-know-from-the-bios-screenshots)
- [Step 1. Capture the exact error message](#step-1-capture-the-exact-error-message)
- [Step 2. Verify SATA/NVMe mode](#step-2-verify-satanvme-mode)
- [Step 3. Check the boot sequence and boot mode](#step-3-check-the-boot-sequence-and-boot-mode)
- [Step 4. Run Dell built-in diagnostics (ePSA) on the SSD](#step-4-run-dell-built-in-diagnostics-epsa-on-the-ssd)

Context: Renewed Dell Latitude 7410. The laptop **powers on normally** and the charger/power delivery are known good. The remaining symptom is:

- On boot, the laptop reports **"No bootable device"**. Changing the boot sequence in the BIOS did not help.

## Conclusion / Findings

**Diagnosis: the SSD is healthy but has no bootable OS installed. The ESP was wiped.** This is a seller-side software defect, not a hardware failure.

Evidence, from the BIOS screenshots walked through in Steps 2–3:

| Check | Result | What it means |
|---|---|---|
| System Information → M.2 PCIe SSD-0 | 512 GB Micron `MTFDKBA512TGW` detected | Drive is physically present and enumerating. Not a dead drive or dead M.2 socket. |
| General → Boot Sequence | One entry: `UEFI: MTFDKBA512TGW-1BP1AABHA, Partition 1`, checked | The correct drive is in the boot list and enabled. No "wrong entry / disabled entry" problem. |
| General → Advanced Boot Options | Contains only `Enable UEFI Network Stack` — no Legacy toggles | BIOS is UEFI-only on this firmware build. "Wrong boot mode" is not possible here. Rules out Legacy/UEFI mismatch. |
| Add Boot Option → File System List | Shows three GPT partitions: HD(1), HD(3), HD(4) | GPT is intact. Partitions exist. Not a corrupted partition table. |
| EFI Boot Selection on FS0 (= HD(1), the ESP) | Empty Directories column; single file `bootx64.efi` at root; no `\EFI\` tree at all | **This is the failure.** A healthy Windows ESP has `\EFI\Microsoft\Boot\bootmgfw.efi`. This drive doesn't. The BIOS finds the ESP but there is no Windows loader inside it. |

**What this means:**

- The "No bootable device" message is behaving correctly. It is not misleading.
- No BIOS setting can fix this — the missing files are on the drive, not in NVRAM.
- The refurbisher shipped a laptop with a wiped/incomplete OS install. A "renewed" Amazon listing is expected to ship with a working OS, so this is a legitimate return-eligible defect.

**Next actions, in order:**

1. **Step 4 (ePSA)** — run this anyway to confirm the drive hardware itself is healthy. Expected result: pass. A clean ePSA report strengthens the "seller shipped an unusable OS, not a broken laptop" framing when talking to JemJem.
2. **Decide the path forward:**
   - **Return to JemJem** — recommended. Evidence bundle: the BIOS System Information photo, the empty-ESP photo (EFI Boot Selection showing FS0 with only `bootx64.efi` and no directories), and the ePSA pass result.
   - **Or reinstall Windows** from a USB installer built with Microsoft's Media Creation Tool, if Rajiv wants to keep the machine. Hardware appears healthy, so a fresh install should work.

## What we already know from the BIOS screenshots

Photos of **BIOS → General → System Information** confirm:

| Item | Value |
|---|---|
| BIOS version | 1.40.1 |
| Service Tag | GSM1693 |
| Manufacture Date | 12/18/2020 |
| Memory | 16384 MB DDR4-2667, Dual channel |
| CPU | Intel Core i7-10610U |
| **M.2 PCIe SSD-0** | **512 GB, model UUNWL0174L4DVZ — detected** |
| M.2 SATA | (none) — normal; the drive is NVMe, not SATA |

**Implication:** the SSD is physically present and enumerating on the PCIe bus. This rules out a dead drive or a dead M.2 socket. The problem is either a **BIOS boot-mode/setting mismatch** or a **missing/corrupt Windows install on the drive**.

Work through the steps in order. Stop at the first step that resolves the problem, and record the outcome of each step — this is the evidence JemJem needs if a return becomes necessary.

---

## Step 1. Capture the exact error message

Photograph the screen showing "no bootable device" (or whatever exact wording appears — Dell has variants: *"No bootable devices found"*, *"No boot device available"*, *"Boot Device Not Found"*). JemJem asked for this specifically.

---

## Step 2. Verify SATA/NVMe mode

The System Information page shown in the BIOS photos is the wrong page for this step — the SATA/NVMe toggle lives in a different left-rail section. To get there:

1. In the BIOS **left-hand rail**, click the entry named **System Configuration** (it's one of the collapsible section headers, partially visible in the photos as the truncated "…tion" label).
2. That section expands to reveal sub-items. Click **SATA Operation** (may read **NVMe Operation** on this 1.40.1 build).
3. The right pane will then show radio buttons — typically **Disabled / AHCI / RAID On**. That's the toggle this step refers to.

Then:

1. Boot into BIOS with **F2** at the Dell logo.
2. Go to **System Configuration → SATA Operation** (labeled **NVMe Operation** on some 1.40.x BIOS builds).
3. Note the current setting. Common values: **AHCI**, **RAID On**.
4. If it is set to **RAID On** and the drive was imaged for AHCI (or vice versa), Windows will not find a bootable partition even though the drive is detected — which matches what we're seeing.
5. Try toggling to **AHCI**, press **F10** to save, and reboot.
   - If Windows now boots → done.
   - If not → return the setting to its original value before continuing.

---

## Step 3. Check the boot sequence and boot mode

Photos of **General → Boot Sequence** confirm:

- The list contains exactly one entry — `UEFI: MTFDKBA512TGW-1BP1AABHA, Partition 1` — and it is checked. `MTFDKBA512TGW` is the Micron 2300 512 GB NVMe, i.e. the same drive shown in the System Information page. So the "is the SSD in the boot list?" question is already answered: **yes**.
- The "Boot List Option" group on this screen is only the **Add / Delete / View** action group. The UEFI-vs-Legacy toggle is **not** on this page on BIOS 1.40.1.

Do these checks instead:

1. **UEFI/Legacy check — not applicable on this BIOS.** Photos of **General → Advanced Boot Options** show only a single option (`Enable UEFI Network Stack`); there are no Legacy Option ROM toggles. This BIOS build is UEFI-only, and the Boot Sequence list already contains one correctly-checked UEFI entry pointing at the SSD. So the "wrong boot mode" hypothesis is ruled out — skip to sub-step 2.
2. **The Windows bootloader is missing from the ESP — this is the failure.** The EFI Boot Selection dialog (photographed) shows FS0 (= HD(1), the ESP) contains **no directories at all** and a single file `bootx64.efi` at the partition root. A healthy Windows ESP contains `\EFI\Boot\bootx64.efi` **and** `\EFI\Microsoft\Boot\bootmgfw.efi` plus a `\EFI\Microsoft\Recovery\` tree. None of that exists here. **Diagnosis: the drive was wiped/reformatted and Windows was never (fully) reinstalled.** This explains "No bootable device" completely — the BIOS finds the ESP but there is nothing bootable inside it.

   Cancel out of Add Boot Option without saving. No BIOS setting will fix this; the fix is either (a) a clean Windows reinstall from a USB installer, or (b) a return to JemJem — a "renewed" listing is expected to ship with a working OS. Run Step 4 anyway to confirm the drive hardware is healthy before choosing which path.

   How to reproduce this check (for the record): **General → Boot Sequence → Add Boot Option**, in **File System List** select the HD(1) entry (on this machine: `PciRoot(0x0)/Pci(0x1D,0x4)/Pci(0x0,0x0)/?/HD(1,GPT,34AF9927-F959-445D-81E9-63FEE999DCB8)`), click the `...` next to **File Name**, and inspect what's in FS0. Healthy = `\EFI\Microsoft\Boot\bootmgfw.efi` exists; this machine = only a root-level `bootx64.efi`.
3. **Try toggling Secure Boot once.** **Secure Boot** is its own top-level node in the left rail, below **Security** — scroll the rail down past "SMM Security Mitigation" to find it. Open **Secure Boot → Secure Boot Enable**, note the current setting, flip it, F10, reboot. A Secure Boot mismatch against the installed image can cause boot failure with no other symptom. Restore the original setting if it doesn't help.

---

## Step 4. Run Dell built-in diagnostics (ePSA) on the SSD

The BIOS reports the drive as present, but presence ≠ health. ePSA reads sectors and exercises the controller.

1. Power off completely.
2. Power on and immediately tap **F12** to enter the one-time boot menu.
3. Select **Diagnostics**.
4. Let the **full test** run — 10–20 minutes. Do not skip.
5. If it reports an error, photograph the code (format: `2000-0xxx`, e.g. `2000-0146` = hard drive read error).

Outcome branches:
- **SSD passes all tests** → drive hardware is fine; the OS install is the problem.
- **SSD fails** → hardware defect despite being enumerated. Photograph the code and send it to JemJem along with the BIOS System Information photos and the Step 1 error photo.

**Recorded outcome: Passes all tests. Hardware is fine**


