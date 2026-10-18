# Codex working instructions — KirIvanov/LGPower

This fork extends the open-source LG Power Android remote. Preserve the upstream copyright notices and AGPL-3.0 license. The upstream repository is https://github.com/Nicsilver/LGPower.

## Scope and priority
- Primary objective: **RUTUBE keyboard automation** for an LG 28LK480U (webOS 3.5, firmware 06.10.60) controlled from a POCO X6 Pro 5G.
- The original remote already works, including the D-pad, ENTER, Touchpad, app shortcuts, and VPN exclusion. Do not regress existing features.
- YouTube, VK Video, network/VPN configuration, router changes, and TV firmware changes are out of scope until explicitly requested.
- Work only on the feature branch, not `master`, until a reviewed PR is approved.
- Follow `docs/RUTUBE_KEYBOARD_PLAN.md` and the corresponding GitHub issues. Implement **MVP / Phase A only** first; stop for hardware feedback before Phases B and C.

## Confirmed physical tests
- Standard webOS text insertion works in the TV's web browser, but the built-in search keyboard in RUTUBE does not accept text from LG Power.
- The RUTUBE keyboard displays the Russian letters `й` and `ц` next to each other in its first row.
- In RUTUBE, with `й` highlighted, ENTER inserts the letter and keeps focus on `й`.
- The transition `й` -> `ц` via RIGHT is plausible but must be validated on the TV (do not claim verified in advance).
- Touchpad moves a pointer and can select RUTUBE UI items.

## Implementation rules
- Reuse `WebOsClient.kt` command transport, especially `pressKey()` / `PointerSession`. Commands should be strictly sequential, cancellable, and paced; avoid blocking the Android UI thread.
- Make delays configurable (initial suggestion: 300 ms) and report transport errors. Do not equate a successful socket send with confirmed TV UI processing.
- Do not hard-code TV IP addresses, pairing keys, tokens, Google account details, or any user secrets in code, issues, tests, or logs.
- A test APK must be installable alongside the official `com.nic.lgpower` app; use a debug-only `applicationIdSuffix` or a distinct debug package ID. Preserve official package behavior for production.
- Keep the first prototype minimal: an on-phone test control for `йц`, clear preconditions, Stop, status, and errors.
- Do not automatically clear fields, switch TV apps, or modify original TV settings during Phase A.
- Use deterministic unit tests for any pure navigation model introduced; building an APK is necessary but insufficient for claiming real-device success.
- Document exactly what was changed, what was tested, what remains unverified, and how to install the debug APK safely.
- Validate with `./gradlew assembleDebug` (project README states JDK 17+). If build infrastructure is missing, describe blockers precisely instead of claiming success.

## Collaboration and completion
- Put code in a dedicated PR from `feature/rutube-keyboard` to the fork's `master` when ready for review.
- Avoid auto-merging or editing upstream `Nicsilver/LGPower` without an explicit request.
- Preserve functionality and user settings; keep the original LG Power installed for fallback.
