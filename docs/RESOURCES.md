# Resources for the original Leap Motion Controller (USB `f182:0003`)

This is the **original** Leap Motion Controller (not the newer "Leap Motion
Controller 2"). Everything below was verified against real hardware on Ubuntu
24.04, 2026-07-13.

## Current state of the Ultraleap ecosystem (why this repo exists)

Ultraleap was dismantled and sold off in 2025 — the hand-tracking business
(including Leap Motion tech) went to **ROLI** in Oct 2025; haptics/XR IP went
to SIM IP. Practical fallout:

- `repo.ultraleap.com` (the official APT repo) is **dead** — DNS doesn't even
  resolve anymore. `apt install ultraleap-hand-tracking` per the old docs no
  longer works *directly* — but the Wayback Machine had the whole apt pool
  archived (snapshot dated 2026-01-24, right before it went dark). That's
  what `install.sh` in this repo uses.
- `developer.leapmotion.com`, `leapmotion.com`, `support.ultraleap.com` are all
  dead (no DNS).
- `www.leapmotiontechnology.com` (the rebranded developer portal) is still up
  and offers a Linux **V2.3.1** download (the pre-Gemini generation, also
  works fine on this hardware), but the free-account signup flow was broken
  when we tried it — that's why we ended up on the Wayback Machine route for
  Gemini instead.
- `github.com/ultraleap` has no installer binaries, only source repos (Unity/
  Unreal plugins, `leapc-python-bindings` source, SteamVR driver, etc.).

## 1. Driver / SDK — Gemini v5.17.1.0 (what `install.sh` sets up)

- Service: `ultraleap-hand-tracking-service` (systemd unit, runs as system user
  `leap`), binary `/usr/bin/leapd`. GUI: `/usr/bin/ultraleap-hand-tracking-control-panel`
  (currently broken, see main README). CLI: `/usr/bin/leapctl`.
- Headers/lib for building against LeapC directly: `/usr/include/LeapC.h`,
  `/usr/lib/ultraleap-hand-tracking-service/libLeapC.so`.
- Config: `/etc/ultraleap/hand_tracker_config.json`.
- **The vendor udev rule is broken as shipped** — see the main README for
  details and the fix (`install.sh` applies it automatically).

### Alternative: V2.3.1 (older, pre-Gemini generation)

- Requires a free account at
  https://www.leapmotiontechnology.com/developer-sub/tracking-software-download
  (signup was broken for us as of 2026-07-13 — may be worth retrying).
- Community systemd/RPM install fixes: https://github.com/ksandom/installUltraleap

## 2. Language bindings / APIs

- **LeapC Python bindings** — the recommended way to script against Gemini.
  Upstream: https://github.com/ultraleap/leapc-python-bindings — we clone from
  our preservation fork instead, https://github.com/ohora23/leapc-python-bindings
  (unmodified, Apache 2.0), in case the `ultraleap` org ever disappears like
  `repo.ultraleap.com` did.
  Full API capability breakdown: [`SDK_CAPABILITIES.md`](SDK_CAPABILITIES.md).
- **C API (LeapC)**: `/usr/include/LeapC.h` + `libLeapC.so` — native C/C++ apps
  can link directly. Sample ships at
  `/usr/share/doc/ultraleap-hand-tracking-service/samples/leapc_main.c`.
- **Unity / Unreal plugins**: https://github.com/ultraleap/UnityPlugin /
  https://github.com/ultraleap/UnrealPlugin
- **LeapJS + WebSocket bridge**: https://github.com/ultraleap/UltraleapTrackingWebSocket

## 3. Raw camera access (bypass the tracking service entirely)

- **LeapUVC** (official) — exposes the two IR cameras as a standard UVC webcam;
  examples in Python/C/MATLAB + OpenCV (ArUco tracking, stereo depth, exposure/
  gain control): https://github.com/leapmotion/leapuvc
  (the one-time firmware unlock utility to enable UVC mode is Windows-only;
  after that it's a plain UVC device on any OS)
  - Community fork using libuvc: https://github.com/wtfuzz/leapuvc
- **OpenLeap** — open driver reverse-engineering project (hardware background:
  Cypress EZ-USB FX3 + 2 IR sensors): https://github.com/openleap/OpenLeap
- **leap-jetson** — 3D hand tracking on the raw UVC stream, without the
  Ultraleap service: Monado's Mercury hand models in TensorRT FP16 on an
  NVIDIA Jetson (aarch64), about 80 fps on an AGX Xavier; also runs on a
  desktop CPU with ONNX Runtime: https://github.com/OomAngel/leap-jetson

## 4. Project ideas with existing code to build from

- Touchless mouse control via gestures:
  - https://github.com/rajandeepsingh13/Touchless-Leap-Motion-Mouse-Python
  - https://github.com/openleap/PyLeapMouse
  - https://github.com/JeevanDrave/Control-Mouse-through-Gesture-using-Leap-Motion
- General gesture → keyboard/mouse input mapper: https://github.com/Zeukkari/leapgim
- Custom gesture recognizer (beyond built-in swipe/tap): https://github.com/cevaris/leap-gestures
- Robotic arm control via hand tracking (Ultraleap → serial → Teensy):
  https://github.com/Edison0215/5DOF-Robotic-Arm-Ultraleap
- Browse more: https://github.com/topics/leap-motion (and `?l=python` filter)

## Open questions / things worth checking

- Does `IMUEvent`/`EyesEvent`/`HeadPoseEvent` ever fire on the original
  controller? These exist in the LeapC API but are presumed to require
  Leap Motion Controller 2-only hardware. Not yet confirmed either way —
  if you test this, please open an issue with your findings.
- Does the official `www.leapmotiontechnology.com` signup flow work for
  anyone currently? If the account route becomes viable again, V2.3.1 becomes
  a simpler (if older) alternative to the Wayback Machine route.
