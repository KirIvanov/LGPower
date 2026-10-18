# RUTUBE Keyboard Automation — product and engineering plan

**Repo:** KirIvanov/LGPower (fork of Nicsilver/LGPower)  
**Target:** LG 28LK480U, webOS 3.5, TV firmware 06.10.60; Android phone POCO X6 Pro 5G.  
**Status (2026-10-18):** planning / hardware validation; **no automation feature implemented yet**.

## Product goal

Allow a user to type a search phrase on Android and enter it into the **already opened RUTUBE TV app**, whose custom on-screen keyboard does not accept LG Power's standard webOS text insertion. Eventually add an optional, reliable **Open RUTUBE search** button that operates in an already opened app without restarting it.

The core remote already works, including Touchpad, D-pad, ENTER, app shortcut, and local-network operation while the phone's VPN is enabled with an exception for LG Power. Preserve all of that.

## Evidence — confirmed vs. hypotheses

**Confirmed by real-device tests:**
1. TV's browser accepts text sent by LG Power keyboard.
2. RUTUBE's custom search keyboard appears inside RUTUBE; it does **not** receive text sent by LG Power via the normal remote keyboard mechanism.
3. VK Video also fails to receive text (not part of this implementation).
4. RUTUBE's first row shows adjacent keys `й ц у к е н г ш щ з х`; source: user-supplied television photograph.
5. Selecting the `й` key and pressing OK enters `й`; highlighted focus remains on `й`.
6. Touchpad in LG Power controls a pointer in RUTUBE.

**Not yet confirmed:**
- Whether RIGHT from `й` reliably highlights `ц`.
- Whether the RUTUBE app responds consistently to successive remote commands sent at a configurable interval.
- Whether an arbitrary search phrase can be input without mapping/validating the full navigation graph.
- Whether RUTUBE has any app-specific deep link or direct command opening search.
- Whether webOS can report the active RUTUBE keyboard focus to the network client.

**Technical explanation (hypothesis):** RUTUBE uses its own text widget/keyboard, not exposed to standard webOS IME. LG Power currently uses `ssap://com.webos.service.ime/insertText` for text. The observed difference between browser and RUTUBE supports this hypothesis, but no access to RUTUBE source has validated the internal implementation.

## Phase A — two-character hardware probe (first Codex task)

**User preconditions** (display them clearly in the Android test UI):
1. TV and LG Power test app are paired and reachable.
2. User manually opens RUTUBE and navigates to the on-screen Russian search keyboard.
3. Search field is empty.
4. User manually highlights `й` (top-left first letter).
5. **Only then** user taps `Тест: йц` on the phone.

**Proposed sequence:** `ENTER` (types `й`), `RIGHT` (selects `ц` if the hypothesis is correct), `ENTER` (types `ц`). Pace each command, e.g., start with 300 ms between them, tune based on real tests. A successful send only confirms network submission; user must verify that the TV actually shows `йц`.

**MVP UI:** an isolated RUTUBE Keyboard Test screen/dialog, start button `Тест: йц`, Stop button, current command/status, editable delay, meaningful errors. No generic text input and no automatic jump to RUTUBE search in Phase A.

**Engineering constraints:**
- Use `WebOsClient.pressKey("ENTER"/"RIGHT")` or a properly managed `PointerSession`, and do not overlap concurrent sequences.
- Execute outside UI thread. Preserve ordering; support cancellation between commands and on screen close.
- Reuse active TV pairing without leaking IP/client keys. Handle disconnection and stop immediately on errors. A new debug application ID may require a separate pairing.
- Ship a standalone, differently identified debug APK so it can coexist with the official LG Power.
- Keep upstream AGPL-3.0 notices.
- Validate compile with `./gradlew assembleDebug`; unit-test pure sequencing and cancellation behavior where practical.
- Do not change TV firmware, alter RUTUBE, or introduce a network server, proprietary API, ads, or paid services.
- Human acceptance: user observes exactly `йц` in RUTUBE. No auto-assertion that the TV has processed events.

**Pass/fail checkpoints:**
- PASS: project builds, debug APK installs alongside existing app, normal remote functionality works, and real TV displays `йц` after sequence from confirmed preconditions.
- PARTIAL: APK builds but real-device test unavailable; request hardware test, do not proceed to Phase B.
- FAIL: focus jumps to another key, events are missed/out of order, or UI freezes; record reproducible behavior and adjust the approach.

## Phase B — phrase entry (after user verifies A)

- Collect text using Android keyboard, optionally OS voice dictation, on the phone.
- Model on-screen keyboard as **explicit navigation graph**, not just a rectangular grid, because arrow movement may skip keys, wrap, or focus differently across rows.
- Validate each transition through TV tests; use BFS shortest-path routing only over observed edges.
- Support supported Russian lowercase letters, spaces, deleting, digits, and switching case/language after validating those keys. Handle unknown characters explicitly.
- Build a dry-run command preview, sequence progress, cancel, safe delays; no automatic removal of existing query without confirmation.
- Do not rely on screen OCR or fragile absolute cursor coordinates as the sole approach.
- Treat site/app UI changes as needing revalidation.

## Phase C — open search while RUTUBE already running (separate task)

Research whether RUTUBE on this webOS version has an official app-specific search deep link or command. If not, investigate context-aware key sequences, based on **known screen state**. Do not treat a generic webOS `SEARCH` permission as proof that a global command enters RUTUBE search. No forced app relaunch, and no navigation from unknown states without safe user confirmation.

## Out of scope

YouTube, VK Video, updating the TV OS or TV apps, router changes, IPTV, remote access over internet, Play Store release, or any changes to upstream repository.

## Developer entry points

- `app/src/main/java/com/nic/lgpower/WebOsClient.kt` — `pressKey()`, `PointerSession.sendKey()`, `sendText()`.
- `app/src/main/java/com/nic/lgpower/MainActivity.kt` — D-pad, keyboard, Touchpad handlers; UI entry point.
- `app/build.gradle` — Android build config.
- `README.md` — JDK and Gradle build instructions.
- `AGENTS.md` — instructions to Codex for this fork.

## User testing

1. Keep the original LG Power installed.
2. Install the fork's debug APK **only from the build produced by the user's own repo**.
3. Pair test app if its separate application ID requires it.
4. Open RUTUBE, select search, clear query, highlight `й`.
5. Trigger `Тест: йц`; photograph/report actual text and highlight position.
6. If result differs from `йц`, cancel, record what happened, and do not implement the full keyboard mapping yet.
