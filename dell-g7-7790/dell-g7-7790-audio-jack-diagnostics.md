# Dell G7 7790 — Audio Jack Diagnostics

How to determine whether the 3.5 mm combo audio jack on the Dell G7 7790 is
physically or electrically broken, and what still works if it is.

---

## 1. Quick answer up front

- **Broken jack ≠ dead laptop audio.** The internal speakers and any
  Bluetooth / USB / HDMI / DisplayPort audio path are completely independent
  of the 3.5 mm jack. If the jack is dead you can still play sound through
  the built-in speakers and pair Bluetooth headphones normally.
- **There is no single "test the jack" command in Windows.** Diagnosis is
  done by combining (a) Dell's built-in ePSA/SupportAssist hardware test,
  (b) Windows sound settings behavior when a plug is inserted, and
  (c) driver/registry checks. Details below.

---

## 2. What the jack actually is

The G7 7790 has a single **3.5 mm combo (CTIA / TRRS) jack** on the left
side. It carries both headphone output and headset-microphone input, and it
is wired through the **Realtek ALC3204 / ALC256** codec on the motherboard.
The jack has a mechanical **jack-detect switch** — a tiny sprung contact
that closes when a plug is inserted. Most "broken jack" symptoms on this
model are actually a stuck or worn jack-detect switch, not a dead codec.

Failure modes, from most to least common:

1. **Jack-detect stuck "inserted"** — Windows thinks headphones are always
   plugged in, so the speakers stay muted even with nothing connected.
2. **Jack-detect stuck "empty"** — plugging headphones in does nothing;
   audio keeps coming out of the speakers.
3. **One channel dead / crackling** — worn spring contacts on tip or ring.
4. **No audio at all on the jack, speakers fine** — codec output path or
   solder joint to the jack has failed.
5. **Full codec failure** — no audio anywhere (rare; speakers would also
   be dead).

Knowing which of these you have narrows the fix.

---

## 3. Diagnostic steps (in order)

### 3.1 Dell ePSA / SupportAssist pre-boot diagnostics

This is the closest thing to a "run a command to test the jack" that
exists for this laptop, and it runs **outside Windows** so drivers can't
lie to you.

1. Power off fully.
2. Power on and tap **F12** at the Dell logo → **Diagnostics**.
3. Let the full ePSA run. When it reaches the audio section it will:
   - play a tone through the internal speakers (verifies codec + amp), and
   - prompt you to plug in headphones and play a tone through them
     (verifies the jack path end-to-end).
4. Any failure prints a **service tag + error code** (e.g. `2000-0333`
   for audio). Write it down — Dell's support site decodes it and it's the
   authoritative signal that the jack/codec hardware is bad rather than a
   driver problem.

If ePSA passes the headphone tone test, the jack is electrically fine and
the problem is in Windows.

### 3.2 Windows-side checks

There is no direct "test jack" CLI, but these commands and UI checks
together tell you what Windows thinks is happening:

```powershell
# List all audio endpoints and their current state.
Get-PnpDevice -Class AudioEndpoint | Format-Table Status, FriendlyName

# List the audio hardware itself (codec, Bluetooth audio, HDMI audio).
Get-PnpDevice -Class MEDIA | Format-Table Status, FriendlyName
```

What to look for:

- A **"Headphones"** endpoint that is `OK` **with nothing plugged in** →
  jack-detect switch is stuck closed (failure mode 1).
- A **"Headphones"** endpoint that never appears even when a known-good
  set of wired headphones is inserted → jack-detect switch is stuck open
  (failure mode 2), or the codec has lost the jack entirely (failure
  mode 4).

Also open **Settings → System → Sound** and physically insert/remove a
known-good 3.5 mm plug while watching the output device list. On a
healthy jack the output should flip between "Speakers" and "Headphones"
within about a second of each action.

### 3.3 Realtek codec / driver sanity

If ePSA passes but Windows still misbehaves, the codec is fine and the
driver stack is lying. Try, in order:

1. **Realtek Audio Console** (Microsoft Store app) → Device advanced
   settings → confirm "Classic" vs "Multi-stream" mode, and check that
   the jack-detect override isn't set to "disable front panel jack
   detection".
2. Device Manager → **Realtek Audio** → Uninstall device *with* "delete
   driver" checked → reboot → let Windows Update reinstall, or install
   the current Dell-signed Realtek package from Dell's support page for
   service tag of this laptop.
3. As a last resort, boot a **Ubuntu live USB** and test the jack there.
   Linux uses ALSA/PulseAudio, a completely different stack. If the jack
   works under Linux, the Windows install is the problem; if it fails
   under both, the hardware is the problem.

---

## 4. If the jack really is broken — what still works

The 3.5 mm jack is only one of several audio output paths. All of the
following are independent of it and will keep working on a G7 7790 with a
physically dead jack:

- **Internal speakers.** Wired directly from the codec to the onboard
  amplifier; they do not go through the jack circuitry. Common gotcha:
  if jack-detect is stuck "inserted" (§2 failure 1), Windows mutes the
  speakers *even though they're fine* — this looks like "no sound
  anywhere" but is fixed by disabling front-panel jack detection or by
  overriding the default output device.
- **Bluetooth audio.** The G7 7790 ships with an Intel Wireless-AC 9560
  (or Killer 1550) M.2 card that includes Bluetooth 5.0. Pair
  headphones/speakers via **Settings → Bluetooth & devices → Add
  device**. Bluetooth audio never touches the Realtek codec's analog
  output stage, so a dead jack is irrelevant.
- **USB audio.** Any USB headset, or a $5 USB-to-3.5 mm dongle, presents
  itself as its own audio device and bypasses the internal codec
  entirely. This is the cheapest workaround if the jack is dead and you
  still want to use wired headphones.
- **HDMI / Mini-DisplayPort audio** to an external monitor or receiver.

**Bottom line:** a broken 3.5 mm jack on this laptop is an inconvenience,
not a loss of audio. Bluetooth headphones will work fine.

---

## 5. Repair options if hardware is confirmed bad

The audio jack on the G7 7790 is **not on the motherboard** — it is on a
small daughterboard (the "audio/USB I/O board") connected by a short
ribbon cable. That makes replacement much cheaper than a mainboard swap.

- Dell part search: look up the service tag on
  `dell.com/support` → *Parts* to get the exact I/O board part number for
  this configuration.
- Typical repair path: power down, remove battery, remove bottom cover,
  disconnect the ribbon from the mainboard, unscrew the daughterboard,
  fit the replacement. Dell's service manual for the G7 7790 documents
  the exact steps and torque values.
- If you don't want to open the laptop, a USB-C or USB-A audio dongle
  (§4) is a permanent-enough fix.
