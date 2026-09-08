# Dell Latitude 7410 — Diagnostic Steps

## Table of contents

- [Conclusion / Findings](#conclusion--findings)
- [What we already know from the BIOS screenshots](#what-we-already-know-from-the-bios-screenshots)
- [Step 1. Capture the exact error message](#step-1-capture-the-exact-error-message)
- [Step 2. Verify SATA/NVMe mode](#step-2-verify-satanvme-mode)
- [Step 3. Check the boot sequence and boot mode](#step-3-check-the-boot-sequence-and-boot-mode)
- [Step 4. Run Dell built-in diagnostics (ePSA) on the SSD](#step-4-run-dell-built-in-diagnostics-epsa-on-the-ssd)
- [Step 5. Boot a known-good USB (positive boot-path test)](#step-5-boot-a-known-good-usb-positive-boot-path-test)
- [Step 6. Dump EFI boot variables from a live USB](#step-6-dump-efi-boot-variables-from-a-live-usb)
- [Step 7. Deeper SSD health (SMART, NVMe self-test, full read scan)](#step-7-deeper-ssd-health-smart-nvme-self-test-full-read-scan)
- [Step 8. NVRAM reset and secondary BIOS toggles](#step-8-nvram-reset-and-secondary-bios-toggles)
- [Step 9. Reseat NVMe, RAM, and check the CMOS coin cell](#step-9-reseat-nvme-ram-and-check-the-cmos-coin-cell)
- [Step 10. BIOS update (only if keeping the machine)](#step-10-bios-update-only-if-keeping-the-machine)
- [Quick decision tree](#quick-decision-tree)
- [Appendix: Sudden-onset boot failure scenarios](#appendix-sudden-onset-boot-failure-scenarios)

Context: Renewed Dell Latitude 7410 purchased from JemJem (Amazon order 113-4900276-7285028, placed Jun 16, 2026). The laptop **powers on normally** and the charger/power delivery are known good. The remaining symptom is:

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

---

## Step 5. Boot a known-good USB (positive boot-path test)

Steps 1–4 prove the internal SSD's ESP is empty. This step proves the *rest of the machine* can still boot something — which is the single most useful piece of evidence for the JemJem return, because it converts "no bootable device" into "machine boots fine from USB; the internal SSD has no OS."

1. On another working PC, use Microsoft's **Media Creation Tool** (or Rufus with an Ubuntu ISO) to build a UEFI-bootable USB stick.
2. Insert into the 7410, power on, tap **F12** for the one-time boot menu.
3. Select the USB entry under **UEFI Boot**.
4. Expected outcome: the Windows installer's language picker (or the Ubuntu GRUB menu) loads.
   - **Boots successfully** → firmware, display, keyboard, RAM path are all fine. Only the internal SSD's contents are wrong. Strong "seller shipped an unusable OS" evidence.
   - **Does not boot from USB either** → problem is broader than a wiped ESP. Continue with Steps 6–9.

---

## Step 6. Dump EFI boot variables from a live USB

Even when the BIOS Boot Sequence page "looks right," firmware NVRAM can hold stale `Boot####` entries pointing at a loader that no longer exists. `efibootmgr` reads what the firmware actually has, not what the BIOS UI summarizes.

1. Boot the Ubuntu live USB from Step 5 → "Try Ubuntu."
2. Open a terminal and run:
   ```
   sudo efibootmgr -v
   ```
3. Look for `Boot####` entries whose `File(\EFI\...)` path references a Windows loader (`bootmgfw.efi`). On this machine, expect either no such entry, or one pointing at a path that no longer exists on the ESP.
4. Also run `lsblk -f` and `sudo ls /boot/efi/EFI/ 2>/dev/null || sudo mount /dev/nvme0n1p1 /mnt && sudo ls /mnt/EFI/` — this independently confirms the ESP's contents from OS-level, corroborating the BIOS-side finding in Step 3.

---

## Step 7. Deeper SSD health (SMART, NVMe self-test, full read scan)

ePSA's pass/fail hides raw wear counters. A drive can "pass" while being at 95% wear — still a return-worthy defect on a "renewed" listing.

From the Ubuntu live USB terminal:

1. **SMART attributes:**
   ```
   sudo smartctl -a /dev/nvme0
   ```
   Key fields:
   - `Percentage Used` — should be low (single digits for a healthy drive; 100 = spec'd endurance consumed).
   - `Available Spare` / `Available Spare Threshold` — spare below threshold = drive near end of life.
   - `Media and Data Integrity Errors` — should be 0.
   - `Unsafe Shutdowns`, `Power Cycles`, `Power On Hours` — sanity-check against "renewed" expectations.
2. **NVMe controller self-test:**
   ```
   sudo nvme device-self-test /dev/nvme0 -s 1     # short (~2 min)
   sudo nvme device-self-test /dev/nvme0 -s 2     # extended (longer)
   sudo nvme self-test-log /dev/nvme0             # read results
   ```
3. **Full read scan** (every sector readable — ePSA only samples):
   ```
   sudo dd if=/dev/nvme0n1 of=/dev/null bs=4M status=progress
   ```
   Any I/O errors here = failing drive, regardless of what ePSA said.

Photograph or save `smartctl` output — this is the strongest technical evidence for a return.

---

## Step 8. NVRAM reset and secondary BIOS toggles

Cheap, reversible firmware-side clean slate:

1. **Restore BIOS defaults.** BIOS → bottom-right **Restore Settings** → choose **BIOS Defaults** (not "Factory Settings" — that's more aggressive). Save with F10, reboot. Occasionally clears a stuck boot variable that survived earlier troubleshooting.
2. **UEFI Boot Path Security.** BIOS left rail → **General → UEFI Boot Path Security**. If set to **Always** and no BIOS admin password is configured, firmware may refuse to hand off to the ESP loader. Set to **Never** temporarily; retry boot.
3. **Verify date/time on a cold boot.** If BIOS reports 2020 (the manufacture date) rather than the correct current date after the machine has been unplugged overnight, the CMOS coin cell is dead — see Step 9.

---

## Step 9. Reseat NVMe, RAM, and check the CMOS coin cell

Rules out marginal physical contact and stale NVRAM caused by a dead RTC battery.

1. Power off, unplug charger, hold power button 15s to drain flea power.
2. Remove the back cover (Torx T5 screws around the perimeter).
3. **NVMe SSD:** unscrew the M.2 retention screw, unseat the stick, reseat firmly, replace the screw. Also confirm the screw is actually present — refurbishers sometimes ship without it, causing intermittent contact.
4. **RAM:** unseat both SO-DIMMs, reseat firmly until the clips latch.
5. **CMOS coin cell (CR2032):** on the 7410 it's a small cell taped to the motherboard with a two-pin lead. If Step 8 sub-step 3 showed a reset clock, replace it. A dead cell can silently drop boot entries between power cycles.
6. Reassemble, power on, retest.

**Optional isolation:** if you have a USB-NVMe enclosure, pull the SSD, drop it in the enclosure, plug into a working PC. Confirms the ESP is empty from a second angle and lets you image the drive before any reinstall.

---

## Step 10. BIOS update (only if keeping the machine)

Skip this if returning to JemJem — a return should ship with factory firmware state intact, and a mid-troubleshoot flash muddies the evidence trail.

If Rajiv decides to keep the laptop:

1. On a working PC, go to Dell Support → enter Service Tag **GSM1693** → Drivers → BIOS. Download the latest (currently newer than 1.40.1).
2. Copy the `.exe` to a FAT32 USB stick.
3. On the 7410, F12 → **BIOS Flash Update** → point at the USB → run.
4. Do this **before** reinstalling Windows — some earlier 1.4x builds had NVMe enumeration quirks that could re-appear post-install.

---

## Quick decision tree

```
"No bootable device"  (SSD confirmed present via BIOS photos)
└── ESP inspection [Step 3, sub-step 2]
    ├── bootmgfw.efi present  → check Secure Boot, BCD, etc.
    └── bootmgfw.efi MISSING  → confirmed on this machine.
                                Drive was wiped; no Windows install.
        └── ePSA [Step 4]
            ├── passes → USB boot test [Step 5]
            │           ├── boots from USB → machine is fine; only the
            │           │                    internal SSD's contents are
            │           │                    wrong. Strongest return
            │           │                    evidence. Optional: Step 7
            │           │                    SMART for wear-level proof.
            │           │                    Either reinstall Windows
            │           │                    from USB, or return to JemJem.
            │           └── USB won't boot → broader firmware issue.
            │                                Steps 6 (efibootmgr), 8
            │                                (NVRAM reset), 9 (reseat +
            │                                CMOS cell). If still failing,
            │                                return to JemJem — this is
            │                                beyond a wiped-ESP defect.
            └── fails → hardware defect. Return to JemJem with the
                        ePSA error code + BIOS photos.
```

---

## Appendix: Sudden-onset boot failure scenarios

The main diagnostic flow above assumes the "renewed" machine arrived with a wiped ESP — a defect present from delivery. This appendix covers the *different* scenario: **a laptop that was booting normally and suddenly stopped.** The evidence looks similar ("No bootable device") but the root causes and remediation diverge, so scan this list before assuming the two failures share a cause.

Group causes by what changed just before the failure — that framing narrows the search fastest.

### A. Nothing obvious changed (spontaneous failure)

1. **SSD wear or controller failure.** NVMe drives fail with little warning once `Percentage Used` approaches 100% or the controller firmware hits a bug. Symptoms: drive still enumerates in BIOS (as it does here), but the filesystem is unreadable, or the drive drops off the bus under load.
   - Confirm with **Step 7** (SMART: `Percentage Used`, `Available Spare`, `Media and Data Integrity Errors`; then extended `nvme device-self-test`).
   - A drive that passes ePSA but shows high wear or nonzero integrity errors is a failing drive.
2. **Dead CMOS coin cell.** On a 2020-build machine (this one), the CR2032 is 5+ years old and near end of life. When it dies, NVRAM `Boot####` entries can be dropped between power cycles, and the RTC resets to the manufacture date on every cold boot.
   - Confirm: check BIOS date/time after the machine has been unplugged overnight. Reset to 2020 = dead cell.
   - Remediation: Step 9 (replace CR2032).
3. **Marginal NVMe seat.** Thermal cycling loosens the M.2 stick over years. The drive intermittently drops off the bus, then eventually enumerates but with a corrupted partition table cache.
   - Remediation: Step 9 (reseat + verify retention screw is present and tight).
4. **RTC/CMOS corruption without cell failure.** Rare, but a brownout or ungraceful shutdown during a firmware write can leave NVRAM in an inconsistent state where `BootOrder` points at nothing.
   - Remediation: Step 8 (Restore BIOS Defaults) + Step 6 (`efibootmgr -v` to inspect and rebuild entries).

### B. After a Windows update or reboot

1. **Windows Update broke the boot configuration.** Feature updates and some cumulative updates rewrite BCD; a mid-update power loss or antivirus interference leaves `bootmgfw.efi` missing or `\EFI\Microsoft\Boot\BCD` corrupt.
   - Confirm: Step 3 sub-step 2 (inspect ESP). If the `\EFI\Microsoft\` tree exists but `BCD` is missing or zero bytes, this is the cause.
   - Remediation: Windows Recovery USB → Command Prompt → `bootrec /rebuildbcd` and `bcdboot C:\Windows /s S: /f UEFI` (where S: is the ESP).
2. **Secure Boot re-enabled by a firmware update, image not signed correctly.** UEFI capsule updates delivered through Windows Update sometimes re-enable Secure Boot; if a third-party loader was installed (dual-boot, Linux, some encryption tools), it now fails signature check.
   - Remediation: Step 3 sub-step 3 (toggle Secure Boot off temporarily to confirm; then either sign the loader or leave Secure Boot off).
3. **BitLocker recovery loop mistaken for boot failure.** Not strictly "no bootable device," but often reported the same way. Firmware or TPM changes trip BitLocker into recovery mode; if the user has no recovery key, the drive looks bricked.
   - Remediation: retrieve key from the associated Microsoft account or organization portal.

### C. After a BIOS/firmware update

1. **BIOS reset boot order to defaults.** Firmware updates commonly reset `BootOrder` to the factory default (typically network → USB → internal). If the Windows boot entry was named uniquely, it may not be re-added.
   - Remediation: Step 6 (`efibootmgr -v` to see if the entry exists), then re-add via BIOS **Boot Sequence → Add Boot Option** pointing at `\EFI\Microsoft\Boot\bootmgfw.efi`.
2. **BIOS update failure mid-flash.** Rare with Dell's capsule updates (they're atomic) but possible if the battery died during flash. Machine may not POST at all, or POSTs but can't find storage.
   - Remediation: Dell BIOS Recovery (hold Ctrl+Esc while powering on, boot recovery image from FAT32 USB).
3. **SATA/NVMe mode reset.** Some BIOS updates flip **SATA Operation** back to a default that doesn't match how Windows was installed.
   - Remediation: Step 2 (toggle AHCI ↔ RAID On to match the installed image).

### D. After physical events

1. **Drop or impact.** NVMe is more shock-tolerant than spinning disks, but the M.2 connector or the SSD's own solder joints can fail. Also check for RAM dislodgment.
   - Remediation: Step 9 (reseat everything). If ePSA fails post-drop, that's a hardware verdict.
2. **Liquid ingress.** Even minor spills can short traces on the motherboard or corrode M.2 pins over days/weeks — failure appears "sudden" but has a delayed cause.
   - Remediation: visual inspection under the back cover for residue/corrosion on the M.2 socket. If present, this is not economically repairable on a 5-year-old laptop.
3. **Thermal damage.** Prolonged use with blocked vents can cook the NVMe controller. Symptom: drive works cold, fails after a few minutes of use.
   - Confirm: boot into BIOS and leave it sitting; watch **General → Battery Information** or a live USB's `sensors` output for CPU/SSD temps. If the SSD disappears from BIOS after warming up, thermal is the cause.

### E. After configuration changes (user- or IT-driven)

1. **Someone toggled Secure Boot, SATA mode, or Boot List Option.** Especially in shared/family machines or after an IT visit.
   - Remediation: Step 8 (Restore BIOS Defaults) to get back to a known baseline, then re-verify Steps 2–3.
2. **Disk cloning or partition tool ran.** Tools like Macrium, Clonezilla, or manual `diskpart` can leave the destination drive without a valid ESP even though data partitions look fine.
   - Confirm: Step 3 sub-step 2 (inspect ESP contents). This is *exactly* the pattern seen on the JemJem machine — which is why "refurbisher cloned incorrectly" is the most likely origin story for the delivered-broken case above.
3. **Group Policy / MDM pushed a change.** Corporate-managed machines occasionally receive policies that lock the boot path (e.g., enforce Secure Boot + measured boot); a non-compliant loader then refuses to run.
   - Remediation: check with IT before local troubleshooting.

### How to use this appendix

For a *sudden* failure, start by asking **"what changed just before this?"** and jump to the matching subsection. If nothing changed (Section A), run **Step 7 (SMART) first** — drive wear is the single most common spontaneous cause on a machine of this age. If a Windows update or BIOS update preceded the failure (Sections B, C), the fix is usually software-side and doesn't require the hardware steps.

### Most likely causes for a sudden-onset failure on this class of machine

Ranked by prior probability when the user reports "it was working, then it stopped" with no physical event and no user-initiated config change:

1. **Section B — Windows Update broke the boot configuration.** Most common by volume. Cumulative updates and feature updates rewrite BCD; mid-update interruption, antivirus interference, or a BCD store already in a marginal state leaves `bootmgfw.efi` unreachable or `\EFI\Microsoft\Boot\BCD` corrupt. Users often don't recall the update because it applied on a prior reboot, not the failing one.
   - **First check:** boot Windows Recovery USB → `bootrec /scanos` and `bootrec /rebuildbcd`. If it finds a Windows install but no valid BCD, this was the cause.
2. **Section C — BIOS/firmware capsule update reset boot state.** Second most common. Dell delivers BIOS updates through Windows Update as UEFI capsules; the user sees a "configuring update" screen and a reboot, then no boot. Typical outcomes: `BootOrder` reset to factory (network/USB before internal), Secure Boot re-enabled, or SATA Operation flipped back to default.
   - **First check:** F2 into BIOS → verify **Boot Sequence** still contains the Windows entry and it's checked; verify **SATA Operation** matches how Windows was installed (usually AHCI on modern Dells, RAID On on some OEM images).

The two are often hard to distinguish without evidence because a BIOS capsule update *arrives via* Windows Update — so "after a Windows update" and "after a BIOS update" describe the same reboot from the user's perspective. Disambiguate by:

- Checking **BIOS → System Information → BIOS Version** and comparing to what it was previously (if known). A version bump = Section C.
- Checking Windows Update history from a recovery environment (`DISM /image:C:\ /get-packages`) once the machine is bootable again.

For the **specific JemJem case documented earlier in this file**, none of Sections A–E apply as *sudden-onset* causes — the failure was present from first power-on, which points at Section E sub-item 2 (refurbisher's cloning tool left an empty ESP) as the origin story.

