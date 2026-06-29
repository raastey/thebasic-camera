# basic — Product Spec (v1)

**basic is a camera for street photographers.** Stills only, iPhone Pro, one frame, one dial. It is built around how a street shooter actually works: you live in **shutter** (freeze or smear motion, set your own ISO, read a real light meter), and when you want a face to separate from the crowd you flip to a one-tap **portrait** at ƒ1.4. That's the whole product. No scene modes, no settings page, no studio rig — the viewfinder is the app.

**Two modes, two intents:**
- **S (street) — the mode you live in.** Manual shutter + manual ISO with a true DSLR-style ±EV light meter. Deliberate, fast, the law is the shutter.
- **A (portrait) — the one-tap escape.** Locked at ƒ1.4, auto-exposed, computational subject separation that stays re-editable in Photos. Tap, frame the person, shoot.

**Capture runs entirely on Apple's native iOS camera stack** (AVFoundation, PhotoKit) — see §4. This document is the single source of truth for what basic is, how it behaves, and what ships in v1.

---

## 0. Design baseline — Argentum Camera

**basic** is built on the UX and visual language of [Argentum Camera](https://argentum.camera): the Red Dot–winning street camera whose interface is "hardly noticeable" and lets composition lead. Argentum is a **black-and-white** app; **basic is a color camera** that borrows Argentum's layout, not its monochrome pipeline. Argentum proved that a single capture screen, real-time graded preview, horizontal tick dial, cream-on-black chrome, and a pill shutter can feel native on large iPhones.

**basic keeps that shell. The twists are color and exposure philosophy.**

| Inherited from Argentum | basic's change |
|------------------------|----------------|
| Black chrome, high-contrast HUD, minimal persistent UI | Same restraint — the frame is the interface |
| 4:3 viewfinder as the hero (width × 4/3, top-aligned) | Same default frame; aspect crop is a later non-goal in v1 |
| Real-time look in preview = what you get in the saved file | **JPEG mode:** WYSIWYG — look baked into the file. **DNG mode:** look is preview-only; saved DNG is untouched |
| Horizontal tick-scale dial below the viewfinder | **Dial controls shutter (S, street) or exposure compensation (A, portrait at locked ƒ1.4)** |
| Bottom row: gallery thumb · look label · pill shutter | Same layout; look label shows Mono / Faded / Neutral |
| Swipe down → look filmstrip | Same gesture; three looks instead of six film presets |
| Tap frame → focus; long-press → AE/AF lock | Same |
| Lens pill (focal length mm) on the viewfinder | Same rear lens pill; **selfie flip** control bottom-left (Argentum pattern) |
| Landscape layout mirrors controls under the thumb | Same adaptive intent (see §7) |
| Instant shutter, ready for the next frame immediately | Same non-negotiable |
| Quiet metadata (location, look) in EXIF when enabled | Same |

| Argentum feature | basic v1 stance |
|------------------|-----------------|
| Black-and-white only (all six looks are monochrome) | **Color-first.** Default look is Neutral (accurate color). Mono is optional, not the product identity |
| Shooting modes: NORMAL, STREET, DEPTH, DBLEXP | **Replaced** by **S (street, manual shutter+ISO)** and **A (one-tap ƒ1.4 portrait)** only |
| EV compensation dial (`0.0 EV`) | **Replaced** by the priority dial — no separate EV dial |
| Six B&W film looks (AA, HCB, IP, GW, YK, DL) | **Reduced** to three curated looks (Neutral, Mono, Faded) |
| Format picker: JPEG / HEIC / TIFF / RAW | **Simplified** to **DNG or JPEG** only — no HEIC, no TIFF, no dual-file write |
| Live Photos | Out of scope (§15) |
| Street mode (whole screen = shutter) | Out of scope |
| Double exposure | Out of scope |
| Settings gear / settings sheet | **Neither.** No settings page, no gear, no swipe-up sheet — every control is on the capture screen or a direct gesture |
| Front-camera flip on viewfinder | **Included** — selfie mode (§12.2); JPEG only on front camera |

**One-sentence pitch:** A calm, color street camera in Argentum's tradition — **live in manual shutter (S)** with a real light meter, **flip to a one-tap ƒ1.4 portrait (A)** when a face needs to pop — shooting **DNG or JPEG** on Apple's native stack, for photographers who think in **shutter speeds**, not scene modes.

---

## 1. What basic is

basic is built for the **street photographer** — someone who thinks in shutter speeds, reacts fast, and wants deliberate control without a cockpit of tabs and sliders. The mental model is a film SLR on the street: set your shutter, set your ISO, read the meter, shoot. Portraits are the exception you flip to, not the default you live in. **Full-color** captures in **DNG or JPEG**, a calm Argentum-style viewfinder, nothing studio about it.

The product splits cleanly into two intents:

| Layer | What the user gets |
|-------|-------------------|
| **Exposure** | **S (street / manual shutter):** dial sets exact exposure duration via `setExposureModeCustom`; ISO is manual (standard stops, tap to cycle); a true ±EV light meter shows the result. **A (portrait):** locked at **ƒ1.4** computational depth, auto-exposed with ±EV compensation on the dial — **JPEG only** on rear. |
| **Archive** | **S-mode (rear):** **DNG** (untouched Apple ProRAW) **or JPEG** (look baked in, WYSIWYG). **A-mode (rear):** **JPEG with embedded depth** — processed still + auxiliary depth/portrait matte for re-editing blur in Photos. **Selfie:** **JPEG only** (no ProRAW on front). |

Everything you need is **on the capture screen** or one direct gesture. No settings page. No onboarding cards. Launch lands in the camera.

---

## 2. Who it's for

**Street photographers** who:

- Shoot stills on iPhone Pro hardware in **DNG or JPEG**, fast and on the move
- Live in **manual shutter** and want to set their own ISO and read a real meter
- Reach for "portrait" only occasionally — and want it to be one tap, not a setup
- Care about **color** when archiving RAW; want WYSIWYG grades when shooting JPEG
- Want tactile dial control with haptic detents, not numeric keyboards
- Value a small set of curated looks over a filter store
- Expect Camera.app-class responsiveness — the native stack is not optional

Not for:

- Casual point-and-shoot users who never touch exposure
- Videographers (no video in v1)
- Users on non-Pro iPhones (no triple-camera ProRAW stack)

---

## 3. Design philosophy

**The frame is the interface** — same conviction as Argentum, different exposure grammar.

1. **One screen — no settings.** Capture is the entire app. There is no settings page, no gear icon, no bottom sheet, no onboarding flow. Inspired by Argentum's on-screen row (flash · aspect · mode), but stripped further: only what you need to shoot is visible and tappable.
2. **Two modes, one street + one portrait.** **S** is the mode you live in — **manual shutter + manual ISO**, the optical / ProRAW lane (DNG or JPEG). **A** is the one-tap **portrait** — locked **ƒ1.4** computational depth (**JPEG only**). Tap the mode letter to switch. There is no third mode; "depth" is just A.
3. **The dial is the product.** In **S**, the tick scale is a shutter-speed ruler — **the law**. What you dial is what shoots; ISO is yours (tap to cycle clean stops) and a real ±EV meter tells you the truth. In **A**, aperture isn't a choice (it's locked ƒ1.4), so the dial becomes **exposure compensation** — the one lever a backlit portrait needs.
4. **Gestures, not menus.** Lens (swipe), look (swipe down), focus, and lock live on the frame. No hidden configuration layer.
5. **The shutter is sacred.** Capture is instant — **responsive capture** and **zero-shutter-lag** on when the SDK allows (~Argentum-class readiness). Up to **eight** shots may be in flight. The UI never blocks the shutter for processing.
6. **WYSIWYG in JPEG mode.** Preview matches the saved JPEG — same Argentum contract. In **DNG mode**, the look is viewfinder-only; the saved ProRAW file stays untouched and full-color.
7. **Color is default.** Neutral look on first launch. The viewfinder shows true color unless the user picks Mono or Faded. basic is not a B&W camera with a color escape hatch — it is a color camera with an optional monochrome look.
8. **Two formats on S; one on A.** Rear **S-mode:** DNG or JPEG (user toggle). Rear **A-mode:** **JPEG only** — depth data and portrait matte cannot ship inside ProRAW. Selfie remains JPEG only. No HEIC sidecar, no TIFF, no dual-file capture.
9. **Honest readouts.** In **S**, the HUD shutter line **matches the dial** — shutter is law; ISO shows the chosen manual stop; the ±EV meter shows true under/over. In **A**, shutter and ISO are ISP-chosen and shown live; the aperture line reads a static `f1.4`; the dial value is the ±EV compensation.
10. **Privacy by default.** No account, no network, no analytics. Photos stay on device and in the user's library.
11. **Native stack first.** Smoothness is a product requirement. **Strictly Apple native camera abilities** for session, preview, exposure, shutter speed, ProRAW, and PhotoKit — custom code only for UI, looks, and off-hot-path file write (see §4).

**Mood:** Argentum-quiet. Pure black chrome. White ink for primary labels. Brass (or Argentum-yellow) for active selection only. Cream pill shutter. SF Mono numerals on the dial and readouts.

---

## 4. Native camera stack

**basic must feel as seamless as Apple's Camera app.** The Argentum shell is custom UI; **capture, exposure, and shutter speed are strictly native** — only what AVFoundation and `AVCaptureDevice` expose on Pro hardware. No third-party camera SDKs. No parallel exposure engine that fights the ISP.

### Product bar

| Expectation | Standard |
|-------------|----------|
| Launch | First live preview frame in **< 1.5 s** cold start |
| Preview | Stable, non-floaty, ProMotion-smooth — no visible wobble from over-correction |
| Shutter | Instant acceptance; **zero-shutter-lag** and **responsive capture** enabled when the SDK supports them |
| Dial | Dial step applies immediately; S HUD shutter equals dial label |
| Lens switch | Minimal blackout; same-sensor zoom keeps preview live |
| Capture → next shot | Ready for the next press while prior writes finish in the background |
| Interruptions | Calls, Control Center, backgrounding — resume without lost state or crash |

If a feature costs native feel, it ships behind a toggle, a thermal gate, or not at all in v1.

### Required Apple APIs and flags

Use the public stack fully — read capabilities at runtime on every supported Pro model:

| Layer | Native path |
|-------|-------------|
| Session | `AVCaptureSession` (`.photo` preset), dedicated **session queue** for all config and capture |
| ProRAW | `AVCapturePhotoOutput` Apple ProRAW pixel formats when DNG mode is active |
| Responsiveness | `isResponsiveCaptureEnabled`, `isZeroShutterLagEnabled` on `AVCapturePhotoOutput` when available |
| Preview | **`AVCaptureVideoPreviewLayer`** as the primary live viewfinder; Metal looks are an overlay path, not a replacement for focus mapping and orientation |
| S-mode exposure | **`setExposureModeCustom(duration:iso:)`** — fixed duration from dial; ISO held at clamped hardware value. **No** `continuousAutoExposure`, **no** `activeMaxExposureDuration`, **no** ISP shutter auto-adjust. `photoQualityPrioritization = .speed` on capture so fusion does not override manual exposure. |
| Shutter-speed dial | Built from **`activeFormat.minExposureDuration`** / **`maxExposureDuration`** at runtime; dial positions are standard stops the device can attain, filtered to the v1 fast cap (§6.1). HUD shutter line = dial label in S-mode. |
| Long / Bulb | **`setExposureModeCustom(duration:iso:completionHandler:)`** for Bulb and any stop beyond streaming `maxExposureDuration` |
| A-mode (computational) | Virtual rear triple-cam; **`inputAperture` fixed at ƒ1.4** at capture (`PortraitCaptureProcessor`); depth + portrait matte embedded in JPEG |
| A-mode exposure | `continuousAutoExposure` + **`setExposureTargetBias`** from dial (±EV comp). Dial does **not** move physical aperture or blur strength. |
| Focus | `focusPointOfInterest`, `focusMode`, `exposurePointOfInterest`; tap maps through `captureDevicePointConverted(fromLayerPoint:)` on the preview layer |
| Lock | Standard AE/AF lock via device APIs; Camera Control half-press when the system exposes it |
| Photo capture | `AVCapturePhotoSettings` built from **runtime** `availableRawPhotoPixelFormatTypes`, codecs, and max photo dimensions |
| Library | `PHPhotoLibrary` / `PHAssetCreationRequest` — DNG or JPEG resource only; stream to disk; never block the shutter |
| RAW decode / grade | `CIRAWFilter` and look render **off the preview hot path** — JPEG bake and thumbnails only after delegate callback |
| Lenses | Runtime `AVCaptureDevice` discovery. **S DNG:** physical camera per lens stop. **S JPEG:** virtual rear triple-cam + full catalog (incl. digital zoom crops on iPhone 17). **A:** virtual device + **native stops only** (§6.2.3) — no intermediate crops or tele digital zoom. See [`IMPLEMENTATION.md`](IMPLEMENTATION.md) §2–§3. |
| Lifecycle | `AVCaptureSession` interruption / runtime-error observers; background task for in-flight writes |

### Architecture rules

1. **Preview path is sacred.** No DNG decode, PhotoKit writes, heavy Core Image graphs, or thumbnail generation on the thread that delivers preview frames.
2. **Main actor stays UI-only.** Session configuration, `capturePhoto`, and device property writes never block SwiftUI.
3. **Honest runtime limits.** The shutter-speed ladder, ISO range, ProRAW availability, and photo dimensions are derived from **`activeFormat`** and device properties at runtime — never hardcoded per iPhone model. If the SDK cannot attain a dial position, that stop is omitted.
4. **Shutter never waits.** Up to **eight** captures in flight; excess queued, never silently dropped. Processing and library I/O run on background queues.
5. **Native before custom.** If Apple ships a fast path (responsive capture, ZSL, preview layer), use it — except in **S-mode**, where manual shutter via `setExposureModeCustom` takes precedence over auto-exposure shortcuts.
6. **Looks are optional cost.** Live Metal grades may simplify or disable under thermal pressure; capture quality and shutter responsiveness do not degrade.
7. **Measure on device.** `os_signpost` (or equivalent) on cold launch → first frame, shutter press → delegate callback, dial step → readout update, lens switch start → first frame. Subjective "feels fine" is not a ship gate.

### What we do not do

- Third-party camera or filter SDKs
- Blocking the shutter until a file finishes writing
- Replacing `AVCaptureVideoPreviewLayer` with a custom preview unless Metal looks require it — and even then keep the native layer for POI mapping
- Per-frame `@Observable` / main-actor state writes from the video delegate
- Continuous horizon/rotation correction that makes the viewfinder feel floaty
- Hardcoded shutter-speed tables per device model (always query the SDK)
- Dial positions the native stack cannot apply — the UI only offers **attainable** speeds
- Custom exposure loops that auto-adjust shutter speed in S-mode (shutter is law — user dials)
- `activeMaxExposureDuration` ceiling semantics in S-mode
- Holding full 48MP DNG buffers in memory longer than necessary

---

## 5. Device requirements

| Requirement | Detail |
|-------------|--------|
| Hardware | iPhone 14 Pro, 14 Pro Max, 15 Pro, 15 Pro Max, 16 Pro, 16 Pro Max, 17 Pro, 17 Pro Max |
| OS | iOS 18+ (ships against current SDK) |
| Display | Portrait-primary; ProMotion assumed for dial and meter animation |
| Optics | Rear multi-camera ProRAW stack required |

Unsupported devices see a static explanation screen. No degraded mode on standard iPhones.

---

## 6. Exposure model

### 6.1 Manual shutter (S) — default

**Shutter is law.** The thumb dial sets the **exact** exposure duration of preview and capture. The ISP does **not** auto-adjust shutter faster or slower than the selected stop. If the scene is too dark at 1/250, the photographer dials to 1/60 or 1/30 — basic does not decide that for them.

**Manual ISO, not auto.** Because the iPhone aperture is fixed glass, true S-mode (which balances by moving the aperture) has no auto variable on iPhone — so ISO is set **manually** from a short ladder of clean full stops and the ±EV meter becomes a real D6-style light meter that moves a full stop per shutter or ISO stop. Technically this is manual exposure framed shutter-first. S-mode uses `setExposureModeCustom(duration:iso:completionHandler:)` — the dialed `CMTime` and the chosen ISO, applied at capture.

**ISO ladder.** Up to six standard full-stop values (50 … 25600) filtered to what the active format can reach, cleanest first — not raw sensor-base multiples. Default = cleanest. **Tap the ISO readout** to step cleaner→dirtiest (wraps); **long-press** steps back. ISO resets to cleanest each launch.

**How it works**

1. **Discover speeds at runtime** from `AVCaptureDevice.activeFormat`:
   - `minExposureDuration` — fastest the format allows
   - `maxExposureDuration` — longest single-frame exposure the hardware supports
2. **Build the dial ladder** as standard stops (1 s, 1/2, 1/4 … 1/2000) **intersected** with what the format can attain. Omit stops below `minExposureDuration` or above the v1 product cap. Never invent speeds the ISP cannot honor.
3. **Apply selection natively:** dial stores `duration`; preview stays in **continuous auto exposure** (no brightness simulation from the dial). At shutter, `setExposureModeCustom` with the dialed duration + the manually chosen ISO.
4. **Readouts:** HUD shutter = **dial label**; the ISO line shows the **chosen manual ISO**; the **±EV meter** (needle + signed value) shows the true under/over of shutter + ISO against the metered scene and moves freely. Preview brightness does not chase the dial — meter, not screen, tells you exposure.
5. **Tap focus** sets focus POI only — does **not** revert S-mode to auto exposure.
6. **Capture:** `AVCapturePhotoSettings.photoQualityPrioritization = .speed` in S so multi-image fusion does not override manual exposure in low light.
7. **Bulb / beyond native max:** same `setExposureModeCustom` path with extended duration; clearly labeled in UI.

**v1 product cap:** fast end of the dial stops at **1/2000 s** even when `minExposureDuration` allows 1/4000 or faster. Slow end runs down to the device's native minimum-duration floor (then Bulb/long if enabled). Example ladder when the format supports it: Bulb → 30 s → … → 1/1000 → **1/2000**. 1/4000 and faster are out of scope until the cap is raised.

**What we do not do in S-mode:** `activeMaxExposureDuration` ceiling semantics, exposure-target bias, **auto ISO**, or any ISP shutter compensation the user did not dial. (Continuous auto exposure **is** used for the preview only — never for capture, which is always the dialed shutter + chosen ISO.)

### 6.1.1 Live metering and low-light stack (S-mode)

The meter is the law of exposure feedback; the preview is for composition only. All tools are **S-mode only**. The ±EV meter is always on; **histogram and zebra each have a toggle icon next to the grid** in the viewfinder bottom row (state persisted). The low-light stack is automatic:

1. **±EV meter** — analog needle scale at the bottom of the finder plus the signed value in the readout. With manual ISO nothing auto-compensates, so it reads the true deviation of (dialed shutter × chosen ISO) from Apple's metered scene reading (`exposureDuration` × `iso`, plus `exposureTargetOffset`) and **moves a full stop per shutter or ISO stop**. Pins to `UNDER`/`OVER` past ±3.
2. **Histogram** — live luminance histogram in the readout, computed CPU-side from subsampled preview buffers (off the GPU, never blocks frame delivery). Top/bottom bins flag amber when highlights/shadows clip.
3. **Highlight zebra** — diagonal stripes composited over blown highlights via the grade-overlay path; transparent elsewhere so the native preview shows through on the Neutral look.
4. **Low-light stack (escape hatch)** — at the **dirtiest (top) ISO stop**, format **JPEG**, and roughly exposed, the shutter fires an `AVCapturePhotoBracketSettings` burst of identical frames at the dialed shutter + chosen ISO; they are averaged (~√N cleaner grain) into one upright JPEG. Shutter + ISO stay exactly as set. A `STACK` chip confirms it. **DNG never stacks** — ProRAW stays an untouched single frame.

### 6.2 Aperture (A) — the portrait lane, locked at ƒ1.4

**Product intent:** A-mode exists for **one job — portraits** — and it is deliberately drastic. There is no aperture ladder and no DoF dial: **A is always ƒ1.4**, the strongest subject/background separation the computational depth pipeline can deliver. A street shooter reaches for "portrait" when they want a face to pop, not to fiddle with f-stops. Tap into A, frame the person, shoot. The blur is baked into a depth JPEG and stays re-editable in Photos.

This mirrors the product split: **S is the street mode you live in; A is the one-tap portrait escape.** (See §1.)

#### 6.2.1 What the dial does (and does not)

| | **S-mode dial** | **A-mode dial** |
|---|----------------|-----------------|
| Controls | **Exact shutter duration** (`setExposureModeCustom`) | **Exposure compensation** (`setExposureTargetBias`, ±EV in ⅓ stops) |
| Aperture | Fixed lens, read-only in HUD | **Locked ƒ1.4** — computational, not selectable |
| Depth of field | Optical only | Fixed strongest computational blur (re-editable in Photos) |
| Brightness | User dials shutter + ISO; meter shows result | Auto exposure; dial nudges it ±EV (preview reflects it) |

Because aperture is no longer a choice, A frees the hero dial for the one lever a portrait actually needs: **exposure compensation** for backlit or high-key faces. The big value reads `+0.3 EV` etc.; the HUD aperture line shows a static `f1.4`. Double-tap the dial resets to `0 EV`.

#### 6.2.2 Format lock — JPEG only in A

Entering **A** on the rear camera:

1. If `format === 'DNG'`, set `rearFormatMemory = 'DNG'` (unchanged) but force **`format = 'JPEG'`** for the active session.
2. **Format label** shows **`JPEG`**; tap is **disabled** (no DNG toggle while A is active).
3. Optional status chip on format tap attempt: *"A uses depth JPEG"* — one line, auto-dismiss; no sheet.

Leaving **A** for **S**:

1. Restore `format` from `rearFormatMemory` (typically DNG if that was the user's rear preference).

**Selfie:** always JPEG in both modes. TrueDepth can supply depth on front; same computational JPEG contract applies when A is active on selfie.

**ProRAW / DNG and computational depth are mutually exclusive.** Raw sensor bytes cannot carry Apple's portrait depth package. A-mode never offers DNG on rear.

#### 6.2.3 Native lenses only (no digital zoom)

In **A-mode**, lens swipe and the lens pill cycle **native stops only** — the same primary lenses the system Camera app exposes, not intermediate wide crops or tele digital zoom:

| Generation | A-mode stops |
|------------|--------------|
| iPhone 14 / 15 Pro Max | 13, 24, 48, 77 mm |
| iPhone 16 Pro Max | 13, 24, 48, 120 mm |
| iPhone 17 Pro Max | 13, 24, 48 mm |

**S-mode** keeps the full catalog (28, 35, 100, 200 mm on iPhone 17, etc.). Switching **S → A** snaps to the nearest native stop if the current lens is not in the portrait set.

#### 6.2.4 Preview and capture-time depth

When `mode === 'A'`:

1. Preview is the **native sharp** `AVCaptureVideoPreviewLayer` (+ optional look grade). **No live depth blur** in the viewfinder — by design: the photographer is fine seeing the blur land in Photos afterward, and it avoids preview churn / session instability.
2. Depth and portrait matte are requested on **shutter** via `AVCapturePhotoSettings`.
3. `inputAperture` is **fixed at ƒ1.4** (`CameraSessionController.portraitInputAperture`) in `PortraitCaptureProcessor` — no per-shot aperture mapping.
4. Saved JPEG is WYSIWYG for look + blur; pixels written **upright**; depth auxiliary embedded for Photos re-edit.

Looks (Neutral / Mono / Faded) apply on the preview grade path; saved A-mode JPEG includes look + capture-time blur.

**Session topology:** A-mode uses virtual rear + depth-capable format; depth blur runs at **capture** time (`PortraitCaptureProcessor`), not as a live preview overlay. S-mode **DNG** uses **physical** wide / ultra-wide / tele inputs for full ProRAW resolution; S-mode **JPEG** uses virtual device + zoom for fast lens handoff. Mode or DNG-format switch triggers session reconfiguration on the session queue. See [`IMPLEMENTATION.md`](IMPLEMENTATION.md) §2, §6.

#### 6.2.4 Capture and re-edit in Photos

A-mode shutter fires **processed JPEG** settings with:

- Active look baked in (WYSIWYG)
- **`isDepthDataDeliveryEnabled`** / depth delivery as SDK allows
- **Portrait effects matte** delivery when supported
- **`embedsDepthDataInPhoto`** so auxiliary depth/matte travel inside the file

The saved asset opens in **Photos** (and compatible editors) with **adjustable Portrait blur** after capture — the same re-edit contract as Apple's Portrait stills. basic does not ship an in-app depth editor.

**Sandbox-only saves** (Photos permission denied) still write JPEG files but **lose** system re-blur in Photos; persistent sandbox banner unchanged (§11).

**DNG:** never written in A-mode. No dual-file RAW+JPEG.

#### 6.2.5 Honest limits

- Effect strength depends on **scene geometry**, subject separation, and hardware — flat scenes and distant subjects produce weak blur (same as system Portrait).
- Portrait effects matte quality is best with **people**; generic depth blur still works but edge quality varies.
- Preview stays sharp; blur is applied at capture — dial intent is visible in the saved file, not as live bokeh in the viewfinder.
- No promise of DSLR-equivalent bokeh on every lens stop or every subject.

### 6.3 Mode switching

- Tap the mode letter (`S` or `A`) on the viewfinder to toggle instantly. Last-used mode persists across launches. Default first launch: **S**.
- Switching **S → A:** force rear `format` to JPEG (§6.2.2); reconfigure for depth capture; the dial recenters to **0 EV**.
- Switching **A → S:** restore `format` from `rearFormatMemory`; reconfigure session for S-mode (ProRAW path when DNG selected).
- Clear `locked` on mode toggle; dial animates to the new mode's `curIdx` (`DESIGN.md` §7).

### 6.4 Metering and lock

- Tap the frame to set focus and exposure point.
- Long-press the frame to lock AE/AF. A lock glyph appears on the frame; `AE/AF` shows in the readout when locked.
- **S-mode:** the ±EV light meter (analog needle + signed value) and optional histogram/zebra are the exposure feedback (§6.1.1). **A-mode:** the dial value *is* the ±EV compensation; a center chip echoes it while dialing.

### 6.5 Manual focus

- Two-finger twist on the frame adjusts manual focus.
- A transient focus scale appears at the bottom of the viewfinder while twisting.

---

## 7. Capture screen

Portrait layout follows the Argentum stack: **4:3 viewfinder** (top) → **horizontal tick dial** → **status strip** → **thumb · look label · pill shutter**. The viewfinder is width × 4/3, top-aligned, filling roughly 60% of the display on Pro Max phones. Chrome fills the remainder with controls anchored to the bottom safe area.

**Landscape:** controls migrate to the thumb side the way Argentum shifts the shutter under the grip — layout follows **`UIInterfaceOrientation`**; dial rotates vertical; shutter column sits under the shooting hand. Same `AVCaptureSession`; only SwiftUI chrome reflows (`DESIGN.md` §3).

```
┌─────────────────────────────┐
│            ●                │   rec indicator (8pt, green)
│ [S]              1/250·ƒ1.78│   mode letter · shutter (dial) · aperture
│                  ISO 400    │
│                             │
│         LIVE FRAME          │   4:3 preview, real-time color look
│         (grid opt-in)       │
│                             │
│  ⟲                 24mm   │   camera flip (selfie) · lens pill (rear)
├─────────────────────────────┤
│         +0.3  |▌|           │   exposure meter (when active)
│      ───|───|───|───        │   priority dial (S: shutter / A: ±EV comp)
│  bolt    4:3  NEUTRAL       │   status strip (flash · grid · look — all tappable)
│  ▢          ◯       DNG/JPEG │   thumb · pill shutter · format label (tap toggles)
└─────────────────────────────┘
```

**Argentum mapping:** the center dial row is where Argentum shows `0.0 EV`; basic shows `1/250` (S) or `+0.3 EV` (A) instead. The row above the shutter is where Argentum shows `HCB` / `GW`; basic shows the active look name. The mode strip replaces Argentum's `NORMAL · STREET · DEPTH` row with display-only status (flash stub, `4:3`, look name) — **S/A lives in the top-left mode letter, not the strip.**

### Viewfinder overlays (on the 4:3 box only)

| Element | Behavior |
|---------|----------|
| Mode letter | Top-left. `S` or `A`. Tap toggles. Brass. |
| Live readout | Top-right. **S:** shutter = dial label; physical aperture; chosen ISO below (tap to cycle); ±EV meter. **A:** live shutter; static `f1.4`; dial = ±EV comp. **"ISO MAX"** amber when at hardware max ISO (A). |
| Rec dot | Top-center. Green 8pt dot. |
| Status chips | Transient center: timer, **low storage**, errors, macro hint, bulb |
| Lens cycle | Swipe L/R on frame (rear only). No separate chip — bottom-left is reserved for camera flip. |
| Camera flip | Bottom-left (Argentum position). Always visible. Tap toggles **rear ↔ selfie**. Circular-arrow icon. |
| Lens pill | Bottom-right (rear only). Opens lens picker. Brass when non-default lens. Hidden in selfie mode. |
| Selfie indicator | Bottom-right in selfie mode. `SELF` label or front-camera glyph. Format locked to JPEG. |
| Lock glyph | Center. Appears when AE/AF locked. |
| Focus scale | Bottom. Appears during manual focus twist. |
| Grid | Tap **`4:3`** in the status strip to toggle rule-of-thirds grid. Off by default. |

### Chrome zone (below viewfinder)

| Element | Behavior |
|---------|----------|
| Exposure meter | Tick needle. Visible **only while dialing** (lingers after release). No EV chip in S-mode. |
| Priority dial | Argentum-scale tick ruler (horizontal portrait / vertical landscape). **S-mode:** manual shutter ladder via `setExposureModeCustom` (§6.1). **A-mode:** ±EV compensation via `setExposureTargetBias` (§6.2; aperture is locked ƒ1.4). Double-tap resets to 1/250 (S) / 0 EV (A). |
| Status strip | **Live controls** (Argentum row pattern, no gear): **flash** (tap) · **`4:3`** (tap toggles grid) · **look name** (tap opens filmstrip). No `NORMAL`/`STREET` modes. |
| Thumbnail | Last shot (Argentum bottom-left square). Tap opens Photos at saved asset. |
| Shutter | Cream pill rounded rect, brass border — same visual weight as Argentum's white pill. Tap fires. Long-press opens self-timer picker. Bulb on long-press when Bulb selected in S-mode. |
| Format label | Bottom-right (rear). **S-mode:** shows `DNG` or `JPEG`; tap toggles; brass when DNG. **A-mode:** shows **`JPEG` only**; tap disabled (§6.2.2). Selfie: **`JPEG`** only, not tappable. |

---

## 8. Gestures and on-screen controls

**No settings page.** Every control is on the capture screen or a direct gesture below.

### On-screen (tap)

| Control | Location | Action |
|---------|----------|--------|
| Mode | Top-left `S` / `A` | Toggle street (manual shutter) vs portrait (ƒ1.4) |
| Format | Bottom-right `DNG` / `JPEG` | **S:** toggle format (rear only). **A:** JPEG locked. Selfie = JPEG locked. |
| Camera flip | Bottom-left viewfinder | Rear ↔ selfie |
| Lens pill | Bottom-right viewfinder (rear) | Lens picker |
| Flash | Status strip | Toggle flash / torch when SDK supports |
| Grid | Status strip `4:3` label | Toggle rule-of-thirds grid |
| Look | Status strip look name | Open look filmstrip |
| Thumbnail | Bottom-left chrome | Open last shot in Photos |
| Shutter | Center pill | Capture; long-press timer / Bulb |

### Gestures

| Gesture | Action |
|---------|--------|
| Swipe down on frame | Open look filmstrip |
| Swipe left / right on frame | Cycle rear lens (disabled in selfie mode) |
| Tap camera flip | Toggle rear ↔ selfie (§12.2) |
| Tap frame | Focus + meter point |
| Long-press frame | Lock / unlock AE/AF |
| Two-finger twist | Manual focus |
| Tap mode letter | Toggle S ↔ A |
| Double-tap dial | Reset active parameter to default |
| Action Button (system) | Cycle look when bound via Shortcuts |

Precedence (lowest to highest): tap < long-press < two-finger twist < edge swipe. Dial drag is captured only within the dial hit area.

**First launch defaults:** S-mode, DNG (rear), Neutral look, grid off. No tutorial cards — straight into the viewfinder.

**Haptics:** dial tick, end-stop, shutter, lock — follow **system haptics** setting. No in-app haptics menu.

**Persistence:** last mode, format, look, grid state, and rear lens restore on next launch via local storage. No UI to "set defaults" — last used is the default.

---

## 9. Hardware controls

| Control | Behavior |
|---------|----------|
| Volume buttons | Shutter. Long-press enables burst (up to in-flight cap). |
| Camera Control | Half-press locks AE/AF. Release captures (when enabled by system). |
| Action Button | Cycle look via `Toggle Look` App Intent (user binds in iOS Settings). |

---

## 10. Looks

Argentum ships six film-inspired **monochrome** presets (AA, HCB, IP, GW, YK, DL) in a swipe-down tile strip. **basic keeps the interaction but is a color camera:** default preview and archive companion are full color; looks grade color or optionally convert to mono.

Three curated looks. **Neutral is the default on first launch.**

| Name | Character | Argentum lineage |
|------|-----------|------------------|
| **Neutral** | Accurate **color**. Default. Product identity. | Not Argentum's B&W baseline — basic shoots color |
| **Mono** | High-acuity monochrome. Optional creative grade. | In the spirit of Argentum's classic B&W grades |
| **Faded** | Muted **color**, warm shadows. | Softer cousin to Argentum's faded-film shoulder |

- Renders **live in the viewfinder** via Metal when Neutral preview is off — **color-accurate by default**.
- **JPEG mode:** look is **baked into the saved file** (WYSIWYG).
- **DNG mode:** look is **preview-only**. Saved ProRAW DNG is untouched full-color sensor data. **Never** forces monochrome on the DNG.
- Swipe down opens a thin filmstrip (Argentum tile row pattern) for quick selection.
- Action Button cycles looks when the shortcut is configured.

---

## 11. Archive

Every shutter press saves **one file** in the active format. No dual writes. No companion files.

### DNG mode (default)

- **Apple ProRAW DNG** — untouched bytes from the SDK. Full **color** sensor data. Opens in Lightroom, Snapseed, Photos RAW editors.
- Active look affects the viewfinder only. The DNG is never modified.
- **S-mode DNG** captures through the **physical** lens camera (wide for 24mm, tele for 100/200mm, etc.) so ProRAW reaches full sensor resolution (~48MP on wide at 24mm). Ultra-wide (13mm) remains ~12MP — hardware limit.
- See [`IMPLEMENTATION.md`](IMPLEMENTATION.md) §4 for dimension expectations per stop.

### JPEG mode (S and A)

- **S-mode JPEG:** processed still with the active look and color grade **baked in**. WYSIWYG. EXIF focal length: catalog mm when zoom matches selected lens; otherwise live zoom at shutter (honest metadata).
- **A-mode JPEG:** same WYSIWYG contract **plus embedded depth and portrait effects matte** when the SDK delivers them (§6.2.4). Pixel data saved **upright**. Photos can re-adjust blur after save.

### Format switching

- Toggle via the **`DNG` / `JPEG`** label on the capture screen (**rear, S-mode only**).
- Format persists across sessions via `rearFormatMemory` when not in A-mode.
- **A-mode:** format label locked to JPEG; `rearFormatMemory` preserved for restore when returning to S.
- In-flight captures complete in the format/mode active when the shutter fired.
- **Selfie mode:** format is always **JPEG**. If the user flips to the front camera while DNG is selected, basic switches to JPEG automatically and restores DNG preference when returning to rear (if the user had DNG selected and is still in S-mode).

If Photos permission is denied, files write to an on-device sandbox with a persistent banner. Thumbnail and shot count still update.

---

## 12. Cameras — rear and selfie

### 12.1 Rear camera (default)

Runtime discovery on rear multi-camera hardware. Focal-length labels (mm) are generation-specific — see `LensFocalLengthProfile` and [`IMPLEMENTATION.md`](IMPLEMENTATION.md) §3.

**S-mode** (full catalog — includes digital zoom stops where the generation exposes them):

| Generation | S-mode stops |
|------------|--------------|
| iPhone 14 / 15 Pro Max | 13, 24, 48, 77 mm |
| iPhone 16 Pro Max | 13, 24, 48, 120 mm |
| iPhone 17 Pro Max | 13, 24, 28, 35, 48, 100, 200 mm |

**A-mode** (native primary lenses only — §6.2.3):

| Generation | A-mode stops |
|------------|--------------|
| iPhone 14 / 15 Pro Max | 13, 24, 48, 77 mm |
| iPhone 16 Pro Max | 13, 24, 48, 120 mm |
| iPhone 17 Pro Max | 13, 24, 48 mm |

Swipe horizontal on the frame to change rear lens (within the active mode's stop list). Focal-length labels flash momentarily. Macro is **S-mode JPEG only** — optional assist banner when focus is near minimum on a non-macro lens; not offered in A-mode.

**Lens routing:**

| Format | Switch mechanism |
|--------|------------------|
| **S DNG** | Physical camera per stop; lens change = session rebuild |
| **S JPEG** | Virtual triple-cam; same-sensor = zoom only; cross-sensor = zoom within virtual device |
| **A JPEG** | Virtual device + zoom; **`availablePortraitStops`** (native mm only); depth configured on A entry |

**DNG and JPEG** both available on rear in **S-mode** when the SDK reports ProRAW support. **A-mode rear captures are JPEG only** (§6.2).

### 12.2 Selfie mode

Argentum places a **camera-flip** control on the viewfinder; basic matches that placement. Selfie is a first-class mode — not a settings afterthought.

**Enter / exit**

- Tap the **camera flip** chip (bottom-left of viewfinder) to switch rear ↔ front.
- Same control returns to rear. Restores the last rear lens and the user's rear format preference (DNG or JPEG).

**Native stack**

- Front camera discovered at runtime via `AVCaptureDevice` (`.builtInWideAngleCamera`, `.front`).
- Same session architecture as rear: native preview layer, S/A exposure paths (§6), tap-to-focus, long-press AE/AF lock.
- Shutter-speed dial uses the front format's **`minExposureDuration`** / **`maxExposureDuration`** — still capped at **1/2000 s** (§6.1).
- Session reconfiguration on flip uses standard `AVCaptureSession` input swap — no custom camera pipeline.

**UI in selfie mode**

| Element | Behavior |
|---------|----------|
| Camera flip | Still bottom-left; returns to rear |
| Lens cycle / lens pill | Hidden — single front camera |
| Selfie indicator | Bottom-right: `SELF` or front-camera glyph |
| Format label | **`JPEG` only** — locked, not tappable |
| S/A dial, looks, shutter | Unchanged — full priority control and WYSIWYG JPEG grade |
| Swipe L/R on frame | No-op for lens (rear only) |

**Archive**

- Selfie captures are **JPEG only**. Apple's front camera does not expose Apple ProRAW on Pro models — basic does not fake DNG on selfie.
- Active look is baked into the JPEG (same WYSIWYG contract as rear JPEG mode).

**Preview**

- Use the system's native front-camera preview behavior (mirrored viewfinder as iOS provides). No extra flip animation that delays the first frame.

**Not in selfie v1**

- Front-camera lens zoom (single fixed front wide)
- Portrait / depth matte on front
- ProRAW or DNG on front camera

---

## 13. Privacy

| Policy | Implementation |
|--------|----------------|
| No network | No URLs, no API calls, no CDN |
| No account | No sign-in, no user ID |
| No analytics | No telemetry SDKs, no crash reporting third parties in v1 |
| Location | No in-app toggle. Written to EXIF only when iOS location permission is granted at the system level. |
| Photos | Library access only to save and hand off. No cloud upload by basic. |

---

## 14. Visual system

**Canonical values:** `tokens/` — `tokens.json`, `Tokens.swift` (`BasicTokens`), `tokens.css`. See `tokens/README.md`. Do not duplicate hex, pt sizes, or durations in Swift views.

### Product rules (not in JSON)

- **Numerals:** SF Mono everywhere a number appears (shutter, ISO, aperture, dial value). Fixed width so values do not jitter.
- **Labels:** SF Pro Text on status strip and filmstrip.
- **Dial / meter:** smooth animation; **120 fps target** on ProMotion hardware. Duration: `motion.dial`, linger: `motion.meterLinger`.
- **Overlays** (grid, level, focus reticle): **fade** in/out (`motion.overlay`). Never slide.
- **Shutter:** scale press (`interaction.shutterPressScale`, `motion.shutterPress`) + soft haptic thunk.
- **Haptics:** dial tick (sharp), end-stop (hard), shutter (soft), lock (confirm). Respects system haptics-off.
- **Layout:** `metric.grid` (8 px) base unit. Tap targets ≥ `metric.minTapTarget` (48 pt). Interactive capture controls in the lower 40% thumb zone (portrait) or shooting-hand column (landscape).

### Token map (summary)

| Key | Use |
|-----|-----|
| `color.chrome` | Background, HUD base |
| `color.viewfinder` | Empty frame behind preview |
| `color.brass` | Active control, mode letter, meter needle, selected state |
| `color.cream` | Shutter body |
| `color.amber` | ISO maxed, under-exposure pin |
| `color.recGreen` | Ready dot |
| `color.ink.*` | Text and lines over frame (92 / 60 / 42 / 24 / 12%) |
| `color.selectedFill` + `metric.selectedBorder` | Filmstrip active tile |
| `metric.shutter.*` | Portrait / landscape pill sizes and border |
| `metric.dialStep` | Priority dial ruler step (25 px) |

Full schema: `tokens/tokens.json`. Implementation contract: `docs/DESIGN.md` §10.

---

## 15. Non-goals (v1)

**Exposure and modes**
- Full manual (M) mode as a *separate* mode (S-mode already does shutter + manual ISO, §6.1; there is no distinct M mode UI)
- Program mode with EV compensation dial (P)
- EV compensation dial alongside the priority dial (Argentum's `0.0 EV` control is not ported — priority replaces it)
- S-mode shutter speeds faster than **1/2000 s** (v1 cap; may expand later)
- Argentum shooting modes: Street (full-screen shutter), Depth, Double exposure

**UI and product shape**
- Settings page, settings sheet, gear icon, preferences screen, or onboarding flow
- Level overlay, focus peaking, neutral-preview toggle, macro-assist banner config (histogram + highlight zebra **are** in S-mode, §6.1.1)
- Separate "Depth" mode or A-mode depth **settings** UI (depth is **built into A**, not a toggle)
- In-app geotag toggle, haptics level picker, "default mode" picker
- Instant post-capture preview sheet (Photos thumbnail is enough)

**Argentum features not ported**
- Monochrome-only product identity (basic is **color-first**)
- Six B&W film looks and named photographer presets
- Live Photos with real-time processing
- HEIC, TIFF, and Argentum's multi-format picker — basic is **DNG or JPEG** only
- Dual-file capture (RAW + processed companion)
- ProRAW / DNG on front camera (selfie is JPEG per native SDK)
- Torch mode (flash control on status strip when SDK supports it)

**General**
- Third-party camera SDKs or non-Apple capture pipelines
- Video, ProRes, log capture
- In-app gallery, editing, social sharing
- Filter store, preset marketplace, subscriptions
- Cloud sync, accounts, AI features
- iPad layout
- Non-Pro iPhone support
- Multiple aspect ratios on capture screen (Argentum's 4:3 / 1:1 / 16:9 row is deferred; 4:3 only in v1)

---

## 16. Launch criteria

basic v1 is ready when:

| Criterion | Target |
|-----------|--------|
| Cold launch to first frame | < 1.5s on target hardware |
| Shutter latency | Zero-shutter-lag path enabled; never blocked by UI |
| Archive integrity | DNG bytes match SDK output (untouched, correct resolution per lens); JPEG reflects active look (WYSIWYG); A-mode JPEG upright; EXIF focal length honest |
| ProRAW resolution | S DNG @ 24mm wide → ~8064×6048 on 16/17 Pro Max; 13mm ultra-wide ~12MP (expected) |
| Burst | Up to 8 in-flight captures without shutter lockup |
| S-mode dial | Ladder built from `activeFormat` exposure durations; `setExposureModeCustom` per stop; fast end at 1/2000 s; HUD shutter equals dial |
| Grid toggle | Tap `4:3` on status strip toggles grid |
| No settings | Zero routes to a settings or preferences screen |
| Live looks | Neutral (color) default in viewfinder; Mono and Faded visible when selected |
| Format toggle | DNG ↔ JPEG on rear in **S**; JPEG locked in **A**; selfie locked to JPEG |
| A-mode depth JPEG | Capture-time blur fixed at ƒ1.4; depth/matte embedded; Photos re-blur works when saved to library; saves upright (§6.2); native lens stops only (§6.2.3) |
| Selfie mode | Flip control works; S/A + looks + native preview; rear state restored on flip back |
| Gesture map | All controls in §8 work; no settings sheet |
| Native feel | §4 product bar met on physical device — preview stable, ZSL on, shutter instant, lens switch measured |
| Device gate | Only supported Pro models run the app |
| Tests | `scripts/verify.sh` passes (build + unit tests) |
| Human UAT | Product director sign-off on physical device before production deploy |

---

## 17. Reference material

| Resource | Purpose |
|----------|---------|
| [Apple AVFoundation](https://developer.apple.com/av-foundation/) | Capture session, photo output, ProRAW, preview layer |
| [argentum.camera](https://argentum.camera) | Design baseline — layout, gestures, WYSIWYG looks, landscape ergonomics |
| App Store screenshots (Argentum) | Tick dial, filmstrip, pill shutter, mode row, EV scale |
| This spec | What basic keeps, changes, and refuses from the baseline |
| [`DESIGN.md`](DESIGN.md) | UI state, layout pixels, control read/write map for engineers |
| [`IMPLEMENTATION.md`](IMPLEMENTATION.md) | Session topology, ProRAW routing, metadata, A-mode pipeline, UAT checklist |
| [`CHANGELOG.md`](../CHANGELOG.md) | Release history |

---

*basic · one screen · no settings · native stack · color · S = ProRAW lane · A = depth JPEG lane · iPhone Pro only*
