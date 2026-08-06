---
description: Flash the Glove80 keyboard firmware. Detects mounted halves and copies glove80.uf2 to them.
argument-hint: [left|right|both] - which half to flash (default: auto-detect)
allowed-tools: [Bash, Read, Glob]
---

# Flash Glove80 Firmware

Flash the compiled `glove80.uf2` firmware onto one or both halves of the Glove80 keyboard.

## Arguments

The user invoked this command with: $ARGUMENTS

Parse the argument as the target half: `left`, `right`, `both`, or empty (auto-detect).

## Instructions

### Step 1: Check firmware exists

Look for `glove80.uf2` in the current directory (`/home/senyc/p/glove80-zmk-config/`). If it doesn't exist, tell the user they need to build the firmware first by running `./build.sh` (requires Docker), or by pushing to GitHub and downloading the artifact from the Actions run.

### Step 2: Detect mounted Glove80 drives

The Glove80 enters bootloader mode (USB mass storage) when you hold the bootloader key. The drive mounts under different paths depending on OS:

- **macOS**: `/Volumes/GLV80LHBOOT` (left) and `/Volumes/GLV80RHBOOT` (right)
- **Linux**: `/media/$USER/GLV80LHBOOT`, `/run/media/$USER/GLV80LHBOOT`, or `/media/GLV80LHBOOT`

Run detection logic:
```bash
# Detect OS
uname -s  # Darwin = macOS, Linux = Linux

# Check for mounted drives (run appropriate for OS)
```

### Step 3: Flash based on argument / detection

- If argument is `left`: flash only the left half (`GLV80LHBOOT`)
- If argument is `right`: flash only the right half (`GLV80RHBOOT`)
- If argument is `both`: flash both halves (user must mount each one in turn)
- If argument is empty: auto-detect whichever halves are currently mounted

To flash a half, copy the .uf2 file to the mounted drive:
```bash
cp glove80.uf2 /Volumes/GLV80LHBOOT/   # macOS example
```
The keyboard will automatically reboot and apply the firmware after the copy completes.

### Step 4: Entering bootloader mode (if drive not found)

If no Glove80 drive is found, guide the user:

1. Hold the **Magic key** (bottom-left corner of the Glove80) to activate the Magic layer
2. Press the **bootloader key** — on the magic layer it's mapped to the `&bootloader` binding (left side: row 4, column 1 — the key that shows `&bootloader` in the magic layer)
3. The half will appear as a USB drive within a few seconds
4. Run `/flash-glove80` again

### Step 5: Report results

- Confirm which halves were flashed
- Note that both halves run the same firmware; you typically only need to flash both when doing a major update or if the halves get out of sync
- Remind the user the keyboard will reboot automatically after flashing

## Notes

- You must flash each half separately — put one half in bootloader mode, flash it, wait for it to reboot, then do the other
- The `glove80.uf2` file flashes both halves identically; there is no separate left/right firmware
- If `./build.sh` fails (Docker not available), the GitHub Actions CI builds it on every push — download the artifact from the Actions tab
