# Vibration Counter

A single-file mobile web tool that uses your device's motion sensors to **detect and count vibrations / taps / impacts in real time** — built for repetitive physical tallying (e.g. counting bacterial colonies by tapping) on an **iPhone or Apple Watch**, straight from the browser, no install.

> **Languages:** English | [中文](README.zh-CN.md)

<p>
  <a href="https://hello-claude.github.io/Vibration-counter/"><img alt="Live Demo" src="https://img.shields.io/badge/Live_Demo-online-2ea44f"></a>
  <img alt="Platform" src="https://img.shields.io/badge/platform-iPhone_Apple_Watch_Web-4c8bf5">
  <img alt="Vanilla JS" src="https://img.shields.io/badge/built_with-vanilla_JS-f7df1e">
  <img alt="Single file" src="https://img.shields.io/badge/single_file-no_build-555">
</p>

🔗 **[Open the live demo →](https://hello-claude.github.io/Vibration-counter/)** — open it on your phone over HTTPS, grant motion access, and tap the big number.

**Highlights**
- 📲 Runs in the phone browser — **no install**, one `index.html`.
- 🎯 High-pass filter cancels gravity/orientation, so only real shakes count.
- 🎚️ Tune sensitivity, upper-limit, and cooldown to match your tapping.
- ⏯️ Tap to reset, long-press to pause; live peak / rate / timer.
- 🔒 Stays awake on iPhone & Apple Watch via multiple anti-lock tricks.

---

## What it does

You open the page on your phone (or Apple Watch), grant motion-sensor access, and tap the big number to start. Every physical vibration that lands inside your configured intensity band increments the count. A **high-pass filter** continuously absorbs gravity and orientation into a slow baseline, so only the *deviation* from that baseline — the actual shake — is measured. You tune three knobs (sensitivity, action upper-limit, cooldown) so it counts the taps you care about and ignores both background jitter and oversized bumps.

It is a **hands-free physical-event tally**, not a scientific accelerometer logger — every figure is a heuristic read from the consumer motion API, not a calibrated measurement.

## How to use

1. **Serve over HTTPS.** Motion sensors are only available in a secure context; open the page via `https://` (e.g. GitHub Pages), not a raw `file://`.
2. **Grant permission.** On iOS 13+ a button appears — tap **授权传感器 / Grant sensor access** and accept the system prompt.
3. **Tap the big number to start.** The status dot turns active and detection begins.
4. **Tap the number again to reset** it to zero (and force-resubscribe the sensor stream); **long-press (~0.6 s) to pause/resume** without losing the count.
5. **Read the live panel:** the accelerometer bar shows current intensity with your lower/upper markers; the three stat cells show **peak (m/s²)**, **rate (counts/min)**, and **elapsed time**.
6. **Quick-convert:** tap any of the `×2 / ×4 / ×8 / ×63.6` buttons to snapshot `count × factor` onto the button; tap again to restore the factor label.

## Controls & display

| Element | Action / Meaning |
| --- | --- |
| Big number | **Tap** = reset to 0 · **Long-press** = pause/resume · also the start trigger |
| Accelerometer bar | Live vibration intensity (0–30 m/s²) with lower-limit & upper-limit markers |
| Peak / Rate / Time | Session peak intensity, counts-per-minute, and elapsed timer |
| `×2 ×4 ×8 ×63.6` | Toggle a `count × factor` snapshot on the button (factors are user-defined, no fixed physical meaning) |

## Settings (灵敏度设置)

| Setting | Range | What it does |
| --- | --- | --- |
| **Sensitivity** | 1–10 | Maps to a lower threshold (1 ≈ 20 m/s², needs a big motion → 10 ≈ 2 m/s², a light shake triggers). |
| **Action upper-limit** | 20–45 | Vibrations stronger than this are treated as oversized impacts and **not** counted; slide to the top (45) for *unlimited*. |
| **Cooldown** | 100–1000 ms | Minimum gap between two counts; continuous shaking is tallied at this cadence. |
| **Current band** | (display) | Shows the active `lower – upper` interval; only vibrations inside it are counted. |

## How it works under the hood

- **High-pass filtering** — an exponential moving-average baseline (α = 0.05) tracks gravity/orientation; the count uses `|magnitude − baseline|`, so tilting the device doesn't false-trigger.
- **Touch guard** — counting is suppressed for ~400 ms after any screen touch, so tapping to reset/pause never registers as a vibration.
- **Stay-awake, belt-and-suspenders** — Screen Wake Lock API + a near-silent 1 Hz AudioContext oscillator + a MediaSession "playing" flag + NoSleep.js, each wrapped in `try/catch` so any unsupported one is silently skipped. This fights iOS/watchOS auto-lock.
- **Stall watchdog** — iOS silently kills the `devicemotion` stream when backgrounded or idle. A 1 s heartbeat, plus `visibilitychange` / `pageshow` / touch hooks, auto-resubscribes the stream when it goes quiet for >3 s, so the counter "wakes up" by itself.
- **rAF-batched UI** — at 60 Hz+ the detection logic runs in the event callback for accuracy, while purely visual updates are coalesced into one `requestAnimationFrame` per frame to keep taps responsive.

## Requirements & compatibility

- **HTTPS / secure context** is mandatory for sensor access.
- **iPhone (iOS Safari)** — fully supported, including the iOS 13+ permission prompt and Wake Lock.
- **Apple Watch (watchOS WebKit)** — targeted; some keep-awake paths (Wake Lock, NoSleep) may not take effect, but counting still works if motion events arrive.
- **Android Chrome** — the standard `devicemotion` path works; some keep-awake tricks vary by vendor.
- No build step, no dependencies to install — a single `index.html` (NoSleep.js loads from a CDN; if it fails, counting is unaffected).

## Limitations

- **Heuristic, not calibrated.** Values come from the consumer motion API; they are not a metrology-grade measurement and vary by device.
- **Threshold-based tallying.** A vibration is counted only if it lands in the `[lower, upper]` band and clears the cooldown — overlapping or sub-threshold events can be missed, and the cooldown caps the maximum count rate.
- **Platform keep-awake is best-effort.** Auto-lock prevention is not guaranteed, especially on watchOS; the screen may still sleep.
- **Sensor stream is fragile on iOS.** Despite the watchdog, aggressive backgrounding can still interrupt the stream until you return to the page.
- **No persistence.** Count, settings, and stats are in-memory only; a reload starts from zero.

## Roadmap / Outlook

- **Persist settings & sessions** (localStorage) so sensitivity/cooldown and the last count survive a reload.
- **Configurable quick-convert factors** instead of the fixed `×2/×4/×8/×63.6`.
- **Export / history** — log per-session counts, rate curves, and peaks for later review.
- **Calibration helper** — a guided routine to pick a sensitivity/threshold for a given tapping style.
- **Installable PWA** — offline manifest + service worker so it launches like a native app.

## License & usage

Copyright © 2026 **hello-claude**. All rights reserved.

This project is owned and maintained solely by its author. **Modifying it without the author's explicit, prior authorization is not permitted.** Any unauthorized or modified copy is unofficial, unsupported, and used **entirely at your own risk**; the author accepts no liability for any such copy. (This mirrors the authorization notice embedded at the top of `index.html`.)
