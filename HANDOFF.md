# HANDOFF: Soil Buddy RE monitor (static app, Dabble link)

Updated 2026-11-07, written BEFORE the app files. Attach this plus the folder to a new chat and say what to change.

## 1. History
- 3D simulator: scrapped 2026-11-04. Do not rebuild.
- Grok built a Vite/React monitor (needs Node, cannot use GitHub Pages). Its UI was too bland.
- Earlier plan (static app, own `$T` protocol, Nordic UART, "not Dabble") never got built. The foundation files it mentions (`js/stats.js`, `js/sim.js`, tests) were NOT in this session, so nothing here depends on them.

## 2. What changed this session
- The controller sketch the user uploaded is `soil_buddy_re_controller.ino` v3.0: **DabbleESP32 Terminal** (`Dabble.begin("Soil Buddy RE")`), text commands, emoji text replies. No `$T` lines, no calibration, no camera code.
- So the app now talks Dabble Terminal over BLE. The old "Not Dabble" decision is superseded for the controller.
- App is one file, `index.html` (no build, no Node). Works on GitHub Pages (HTTPS, needed for Web Bluetooth). Phone-first PWA (user said: for app, not laptop): `manifest.webmanifest`, `sw.js`, `icon-192.png`, `icon-512.png`; installs from Chrome on Android after it is hosted over HTTPS. Layout is a phone column (max 430 px) even on a laptop. A real APK would need PWABuilder or Capacitor; not done.

## 3. UI references and what was taken
Mockups: olive hero + weather-style stat card, ring gauge over a field photo, rounded cream cards, dark pill nav. Quick look at common farm/IoT dashboards: one big number first, status = colour + icon + word, tank drawn as a bucket, one-thumb controls.
Built: olive hero (greeting, date, link chip), soil ring gauge, stat card, bucket for the tank, raised centre Water tab, 5 tabs (Home, Camera, Water, Terminal, Setup).

## 4. Protocol
Commands the sketch accepts: `help, soil, water, status, chat on|off, pump test, pump off, pump auto, threshold`.
Lines the app parses (regex in `parse()`): `Soil: 62.0% relative index (ADC n)`, `Water: ~250 mL / 500 mL (ADC n)`, `Pump mode:`, `Pump state:`, `Watering demand:`, `Water guard:`, `Raw ADC:`, `HW-038 ADC:`, `PUMP OFF`, `STARTED`. If the sketch wording changes, change `parse()`.
App sends `status` every 5 s while connected.
Dabble frame used (ASSUMED, unverified): `FF, ver(01), module, function, argc, len..., data..., 00`. Module/function/version are editable in Setup. The app also learns the module id from the first text frame the board sends. Raw hex shows in Terminal (tick "hex").
BLE service: name filter `Soil Buddy RE`; tries FFE0 and Nordic UART, plus an extra UUID typed in Setup. The first notify + write characteristics found are used.

## 5. Status
Built: `index.html`, PWA files, this file. Camera sketch not opened or changed.
NOT verified: Dabble UUIDs and frame bytes (GitHub source blocked automated fetch), real BLE, real camera, the sketches on boards. Demo device only exercises the parser. Say "expected to work", not "tested".
Next step if commands get no reply: open Terminal with hex on, send `status`, copy the hex lines of any reply into a new chat (or paste DabbleESP32 `src/` TerminalModule + Dabble.cpp) and lock the codec.

## 6. Decisions (do not re-ask)
Soil % = uncalibrated relative index, mL = uncalibrated estimate (the app says so). No pH/temperature/growth readings. Pump test is 30 s, supervised, bypasses the water guard (app asks to confirm). Auto is OFF at boot. English only. Demo data is labelled DEMO.

## 7. Known limits
Chrome/Edge on Android, Windows, Mac, ChromeOS only; no iPhone/Safari. BLE needs the page open and in range. Camera from an HTTPS page may be blocked by the browser (mixed content / local network); opening the app from `http://localhost` or a local file avoids it. No CSV export or history storage yet.
