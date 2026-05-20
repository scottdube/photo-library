# Smart Frame Replacement — Project Briefing

> Drop this whole file into a new conversation as context. It's
> standalone and assumes the next Claude has none of the prior
> conversation.

## Goal

Replace the internal electronics of a misbehaving WiFi digital photo
frame while keeping the existing LCD panel and enclosure. Build a
better, more reliable system around the same screen — ideally one that
can pull images from my own self-hosted semantic photo index rather
than a vendor cloud app.

## The hardware in question

**Humblestead 32GB Smart WiFi Frame, 21.5" model (PF2152 family).**

Published specs from the product listings:

- 21.5" IPS LCD, 1920×1080, 16:9
- 32GB internal storage + SD card slot
- WiFi (2.4GHz only, inferred from listings — not stated as dual-band)
- Companion mobile app: AiMOR
- Formats: JPG / JPEG / BMP / PNG, MP4 video up to 15 seconds
- Non-touch on this size (touch is on the 15.6" variant)

Sources:
- https://manuals.plus/humblestead/pf2152-21-5-inch-smart-wifi-digital-photo-frame-manual
- https://www.amazon.com/Digital-Picture-Touchscreen-Humblestead-Instantly/dp/B0BGKX43VH

No teardown of this specific frame appears to exist online.

## Symptom history

1. **Earlier problem (solved):** chronic WiFi disconnects. Resolved via
   networking workarounds (specific fixes not recorded here — UDM Pro /
   2.4GHz environment tuning).
2. **Current problem (unsolved):** frame freezes on a single image and
   stops advancing. UI is unresponsive. Only fix is pulling the power
   and rebooting. After reboot it runs for a while, then locks again.
   Pattern repeats indefinitely.

The freeze-then-power-cycle signature is the canonical behavior of an
Android-based frame with a memory leak or wedged system service on
undersized RAM. Vendor firmware updates rarely arrive for this class of
device.

## What's probably inside (all inference — confirmed only after teardown)

Based on the category and price point, not on this specific unit:

- **SoC:** Rockchip (RK3128 / RK3326 / RK3566) or Allwinner (A40i, A64,
  A133) — quad-core Cortex-A7/A35-class, running a stripped Android
  7.x–9.x.
- **RAM:** 1GB DDR3, possibly 2GB on a 21.5" 1080p unit.
- **Storage:** 32GB eMMC soldered to the main board.
- **WiFi:** Realtek RTL8188-class single-band 2.4GHz module — matches
  the earlier flakiness.
- **Display interface:** standard 21.5" 1080p **LVDS** panel, the same
  family used in cheap PC monitors. Single-channel LVDS, 30-pin FFC is
  the typical combo at this size and resolution.
- **T-CON:** integrated into the panel module on most 21.5" units, so
  the main board only drives LVDS data + backlight enable + brightness
  PWM.
- **Backlight:** separate small LED driver PCB.

## Why replacement is feasible

If the panel is a standard LVDS panel (very likely at this size/res),
it can be driven by off-the-shelf controller boards or by an SBC like
a Raspberry Pi via an HDMI-to-LVDS adapter. Three viable paths:

**Option A — generic LVDS controller board (~$25–40)**
Boards like M.NT68676 or RTD2660H, configured for the specific panel,
accept HDMI/VGA/USB input. A Raspberry Pi 4 or 5 over HDMI then runs
the slideshow. Easiest path; zero custom electronics. Best
starting point.

**Option B — Pi-direct LVDS**
Geekworm or Waveshare HDMI-to-LVDS adapter sized for the panel.
Cleaner mechanically, but adapter has to match the panel exactly.

**Option C — SBC with native LVDS**
Rockchip RK3588 SBCs (Radxa Rock 5, Orange Pi 5) drive LVDS directly.
Overkill for a frame; useful if silent integration matters.

## The interesting downstream play

I'm separately building a local semantic photo search project
(`photo-library`, CLIP + sqlite-vec + FastAPI over WireGuard). The
frame replacement is a natural endpoint client:

- Frame runs **Immich Kiosk** (or a custom equivalent) on a Pi
- Points at *my* FastAPI query service, not Immich
- Rotates through semantic queries instead of chronology:
  "garage / shop photos from last 6 months", "winter landscapes",
  "family + Christmas", etc.

This is the part that justifies the project over just buying a better
frame.

## Information needed to proceed

Before recommending specific controller boards / adapters, I need to
open the frame and capture:

1. **Panel model number** — sticker on the *metal back of the LCD
   module itself*, not the plastic housing. Format like
   `BOE HV215WHB-N00`, `LG LM215WF3`, `AUO M215HW01`, `Innolux V215H1-LE1`.
2. **LVDS connector** — confirm 30-pin FFC and pinout against panel
   datasheet (panelook.com or equivalent).
3. **Backlight driver board** — voltage, current, connector type. Goal
   is to reuse it if possible.
4. **Original power supply spec** — voltage and current on the brick
   label. Need to confirm we can repower the panel + new brain off the
   same supply, or whether a new supply is required.
5. **Photos** — main board both sides, panel back sticker, backlight
   driver, power input area, all connectors with their FFC pin counts
   visible.

## Practical caveats to keep in mind

- **Power:** likely 12V brick; measure before reusing. A Pi 5 + panel
  + backlight pulls real wattage.
- **Mechanical fit:** new controller has to land inside the existing
  housing. Easier at 21.5" than smaller sizes.
- **Touchscreen:** PF2152 (21.5") is non-touch per the manual, so this
  isn't a concern on this specific unit. (If a later swap involves the
  15.6" touch variant, the touch controller is a separate device.)
- **IR remote:** lost in the swap. Use a phone web UI or add a USB IR
  receiver if needed.
- **Firmware option not yet exhausted:** before tearing it down, check
  Humblestead's site for a current firmware update and try a hard
  reset via the rear pinhole (10s hold while powered).

## Working style preferences (for the next Claude)

- Direct, precise, no fluff. Cite source file/URL when stating a fact.
- Flag inference vs. fact explicitly.
- Don't invent UI steps, part numbers, pinouts, or flags. If unsure,
  search first or say so.
- Respect stated constraints (dimensions, materials, hardware).
- Anticipate downstream issues (fit, tolerances, supply current, panel
  lane count, backlight inverter compatibility) rather than answering
  only the literal question.
- Code blocks must contain only executable code — no inline comments,
  no `#`-prefixed annotations. Put labels and explanation in prose
  *outside* the fence. Multi-step procedures use multiple separate
  fences with prose between them.

## Maker context (for the next Claude)

Strong hands-on capability across CNC (Tormach 1100MX), laser (xTool
P2S, F1 Ultra), electronics + embedded (ESP32 family, current vacuum
gauge controller project with GC9A01 display and MPXV6115VC6U sensor),
Fusion 360 incl. Electronics, Arduino ecosystem, woodworking, general
DIY. Comfortable with soldering, FFC handling, oscilloscope-level
debugging. Bias toward understanding the system end-to-end and
designing for repeatability, not one-off hacks.

## First action when starting the new conversation

Open the frame. Photograph the panel sticker and both sides of the
main board. Post the panel part number and photos. From there, the
next conversation can recommend a specific controller board or
adapter rather than category-level guidance.
