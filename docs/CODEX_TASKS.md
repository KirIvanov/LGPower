# Codex work queue — RUTUBE search on LG Power

Status: **ready for Codex Phase A**. GitHub Issues are currently disabled in this fork; this file is the task backlog until they are enabled under Repository Settings → General → Features → Issues.

All work must be performed on `feature/rutube-keyboard` (not `master`), with the constraints in [AGENTS.md](../AGENTS.md) and [RUTUBE_KEYBOARD_PLAN.md](RUTUBE_KEYBOARD_PLAN.md).

## TASK-001 — MVP A: two-character `йц` keyboard probe [READY]

**Objective:** In already-open RUTUBE search with Russian virtual keyboard visible, and starting on `й`, automate three TV control commands: `ENTER`, `RIGHT`, `ENTER`.

**User-confirmed state:** highlighting `й` and pressing OK inserts it and leaves focus on `й`. The RIGHT transition to `ц` and complete sequence are unverified, even though the keys are adjacent on screen.

**Codex deliverables:**
1. Add minimal isolated `RUTUBE Keyboard Test` UI accessible from LG Power without breaking existing controls. Explain preconditions to the user.
2. Button `Тест: йц` sends `ENTER` → `RIGHT` → `ENTER` sequentially with initially 300 ms spacing (configurable).
3. Status/progress, transport errors, Stop button, cancellation, no UI thread blocking, no overlapping sequences.
4. Use `WebOsClient` / pointer input and proper resource management. No blind assumptions about TV state.
5. Make debug APK installable alongside existing LG Power using debug-only application ID suffix; preserve normal release app ID.
6. Run Gradle build `./gradlew assembleDebug` and focused tests. State build blockers accurately.
7. Describe changed files, APK retrieval, install/pairing, hardware test checklist, and known limits.
8. Do **not** begin TASK-002 without human confirmation of successful entry of `йц`.

**Acceptance:** a real user reports exactly `йц` appeared in RUTUBE, with original remote unaffected. A successful build is not enough.

## TASK-002 — Phase B: phrase entry via RUTUBE keyboard [BLOCKED BY TASK-001]

- Collect phrase typed or dictated on Android.
- Map RUTUBE virtual keyboard to an explicitly validated navigation graph, including unusual row transitions and special keys.
- Calculate shortest safe paths (BFS) only using tested edges. Preview command count, progress, Stop and errors.
- Russian lowercase + repeated letters first; add space, backspace, numbers, case switching once validated.
- No arbitrary screen taps relying on photographic pixel coordinates as the only solution. No automatic query clearing.
- Add deterministic unit tests and repeatable human acceptance tests.

## TASK-003 — Phase C: open search without reopening RUTUBE [BLOCKED BY INVESTIGATION]

- Research official app-specific search deep link or supported command for the running RUTUBE webOS app.
- If unavailable, derive/test safe sequence from a known UI state, with manual fallback.
- Add one `Поиск RUTUBE` button **inside LG Power** that opens the search of the already-running TV app.
- Never restart the app just to search, never interfere with current playback without user consent.
- Test on user's LG 28LK480U webOS 3.5.

## Non-goals for this phase

YouTube, VK Video, ISP/VPN changes, TV OS upgrades, Android Play Store release, any remote cloud server, and changes to the original Nicsilver/LGPower project.

## Test evidence to request from the user after TASK-001

- Confirm whether the debug build installs in parallel with LG Power.
- RUTUBE search on TV is open, text field empty and `й` highlighted.
- Start `Тест: йц` on phone. Report resulting text, focus state, command delay setting, any error.
- If not `йц`, stop and investigate before expanding the automator.
