# basic — Product Spec (v1)

**basic** is a stills-only manual camera for iPhone Pro models. One frame, one dial, two priority modes. The viewfinder is the app. **Capture runs entirely on Apple's native iOS camera stack** (AVFoundation, PhotoKit) — see §4.

This document is the single source of truth for what basic is, how it behaves, and what ships in v1.

---

## 0. Design baseline — Argentum Camera

**basic** is built on the UX and visual language of [Argentum Camera](https://argentum.camera): the Red Dot–winning street camera whose interface is "hardly noticeable" and lets composition lead. Argentum is a **black-and-white** app; **basic is a color camera** that borrows Argentum's layout, not its monochrome pipeline. Argentum proved that a single capture screen, real-time graded preview, horizontal tick dial, cream-on-black chrome, and a pill shutter can feel native on large iPhones.

**basic keeps that shell. The twists are color and exposure philosophy.**

| Inherited from Argentum | basic's change |
|------------------------|----------------|
| Black chrome, high-contrast HUD, minimal persistent UI | Same restraint — the frame is the interface |
| 4:3 viewfinder as the hero (width × 4/3, top-aligned) | Same default frame; aspect crop is a later non-goal in v1 |
| Real-time look in preview = what you get in the saved file | **JPEG mode:** WYSIWYG — look baked into the file. **DNG mode:** look is preview-only; saved DNG is untouched |
| Horizontal tick-scale dial below the viewfinder | **Dial controls shutter (S) or aperture intent (A), not EV compensation** |
| Bottom row: gallery thumb · look label · pill shutter | Same layout; look label shows Mono / Faded / Neutral |
| Swipe down → look filmstrip | Same gesture; three looks instead of six film presets |
| Tap frame → focus; long-press → AE/AF lock | Same |
| Lens pill (`1×`, `0.5×`, etc.) on the viewfinder | Same rear lens pill; **selfie flip** control bottom-left (Argentum pattern) |
| Landscape layout mirrors controls under the thumb | Same adaptive intent (see §7) |
| Instant shutter, ready for the next frame immediately | Same non-negotiable |
| Quiet metadata (location, look) in EXIF when enabled | Same |

| Argentum feature | basic v1 stance |
|------------------|-----------------|
| Black-and-white only (all six looks are monochrome) | **Color-first.** Default look is Neutral (accurate color). Mono is optional, not the product identity |
| Shooting modes: NORMAL, STREET, DEPTH, DBLEXP | **Replaced** by S-priority and A-priority only |
| EV compensation dial (`0.0 EV`) | **Replaced** by the priority dial — no separate EV dial |
| Six B&W film looks (AA, HCB, IP, GW, YK, DL) | **Reduced** to three curated looks (Neutral, Mono, Faded) |
| Format picker: JPEG / HEIC / TIFF / RAW | **Simplified** to **DNG or JPEG** only — no HEIC, no TIFF, no dual-file write |
| Live Photos | Out of scope (§15) |
| Street mode (whole screen = shutter) | Out of scope |
| Double exposure | Out of scope |
| Settings gear / settings sheet | **Neither.** No settings page, no gear, no swipe-up sheet — every control is on the capture screen or a direct gesture |
| Front-camera flip on viewfinder | **Included** — selfie mode (§12.2); JPEG only on front camera |

**One-sentence pitch:** Argentum's calm capture shell — **in color**, with S/A priority — shoot **DNG or JPEG**, for photographers who think in **shutter speeds and f-stops**, not EV steps and scene modes.

---

## 1. What basic is

basic is built for photographers who think in shutter speeds and f-stops. They want deliberate exposure control, **full-color** captures in **DNG or JPEG**, and a calm viewfinder in the Argentum tradition — not a cockpit of tabs, sliders, and filter drawers, and not a monochrome-only camera.

The product splits cleanly into two layers:

| Layer | What the user gets |
|-------|-------------------|
| **Exposure** | Shutter-priority (S) or Aperture-priority (A). One horizontal ruler dial. Native continuous AE / auto-expose paths (§4, §6). Live meter. Tap, lock, and manual focus on the frame. |
| **Archive** | Every capture saves **one file** in the active format: **DNG** (untouched Apple ProRAW, rear camera only) **or JPEG** (processed still with look baked in, WYSIWYG). Selfie mode is **JPEG only** — native front camera does not offer ProRAW. |

Everything you need is **on the capture screen** or one direct gesture. No settings page. No onboarding cards. Launch lands in the camera.

---

## 2. Who it's for

Photographers who:

- Shoot stills on iPhone Pro hardware in **DNG or JPEG**
- Care about **color** when archiving RAW; want WYSIWYG grades when shooting JPEG
- Know what S-mode and A-mode mean on a real camera
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
2. **Two modes only.** S and A replace Argentum's NORMAL / STREET / DEPTH / DBLEXP row. Tap the mode letter on the frame to switch.
3. **The dial is the product.** Argentum's horizontal tick scale becomes a shutter-speed ruler (S) or aperture-intent ruler (A). This is the single control surface for exposure intent — not a secondary adjustment under a scene mode.
4. **Gestures, not menus.** Lens (swipe), look (swipe down), focus, and lock live on the frame. No hidden configuration layer.
5. **The shutter is sacred.** Capture is instant — **responsive capture** and **zero-shutter-lag** on when the SDK allows (~Argentum-class readiness). Up to three shots may be in flight. The UI never blocks the shutter for processing.
6. **WYSIWYG in JPEG mode.** Preview matches the saved JPEG — same Argentum contract. In **DNG mode**, the look is viewfinder-only; the saved ProRAW file stays untouched and full-color.
7. **Color is default.** Neutral look on first launch. The viewfinder shows true color unless the user picks Mono or Faded. basic is not a B&W camera with a color escape hatch — it is a color camera with an optional monochrome look.
8. **Two formats only.** DNG or JPEG. No HEIC sidecar, no TIFF, no dual-file capture.
9. **Honest readouts.** The HUD shows what the device is actually delivering — live shutter, physical aperture, current ISO — not wishful dial labels alone.
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
| Dial | Device exposure updates feel immediate; readout tracks delivered values |
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
| S-mode exposure | `.continuousAutoExposure` + **`activeMaxExposureDuration`** set from the dial — Apple's native shutter-speed ceiling API; ISP sets ISO. No custom ISO hunt loop. |
| Shutter-speed dial | Built from **`activeFormat.minExposureDuration`** / **`maxExposureDuration`** at runtime; dial positions are standard stops the device can attain, filtered to the v1 fast cap (§6.1). Live value from **`exposureDuration`**. |
| Long / Bulb | **`setExposureModeCustom(duration:iso:completionHandler:)`** only when the selected speed exceeds native `maxExposureDuration` — still AVFoundation, not a side-channel timer |
| A-mode exposure | `.autoExpose` + `setExposureTargetBias` from aperture-intent dial |
| Focus | `focusPointOfInterest`, `focusMode`, `exposurePointOfInterest`; tap maps through `captureDevicePointConverted(fromLayerPoint:)` on the preview layer |
| Lock | Standard AE/AF lock via device APIs; Camera Control half-press when the system exposes it |
| Photo capture | `AVCapturePhotoSettings` built from **runtime** `availableRawPhotoPixelFormatTypes`, codecs, and max photo dimensions |
| Library | `PHPhotoLibrary` / `PHAssetCreationRequest` — DNG or JPEG resource only; stream to disk; never block the shutter |
| RAW decode / grade | `CIRAWFilter` and look render **off the preview hot path** — JPEG bake and thumbnails only after delegate callback |
| Lenses | Runtime `AVCaptureDevice` discovery (rear multi-cam + front wide); zoom on shared rear sensor before physical input swap |
| Lifecycle | `AVCaptureSession` interruption / runtime-error observers; background task for in-flight writes |

### Architecture rules

1. **Preview path is sacred.** No DNG decode, PhotoKit writes, heavy Core Image graphs, or thumbnail generation on the thread that delivers preview frames.
2. **Main actor stays UI-only.** Session configuration, `capturePhoto`, and device property writes never block SwiftUI.
3. **Honest runtime limits.** The shutter-speed ladder, ISO range, ProRAW availability, and photo dimensions are derived from **`activeFormat`** and device properties at runtime — never hardcoded per iPhone model. If the SDK cannot attain a dial position, that stop is omitted.
4. **Shutter never waits.** Up to three captures in flight; excess queued, never silently dropped. Processing and library I/O run on background queues.
5. **Native before custom.** If Apple ships a fast path (responsive capture, ZSL, preview layer, continuous AE), use it before building parallel machinery.
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
- Custom exposure loops that fight the ISP in S-mode
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

### 6.1 Shutter-priority (S) — default

**Strictly native.** S-mode uses Apple's public exposure APIs only. The dial is a UI over the shutter-speed options the device actually supports — not a fictional speed list.

**How it works**

1. **Discover speeds at runtime** from `AVCaptureDevice.activeFormat`:
   - `minExposureDuration` — fastest the format allows
   - `maxExposureDuration` — longest single-frame exposure the hardware supports
2. **Build the dial ladder** as standard stops (1 s, 1/2, 1/4 … 1/2000) **intersected** with what the format can attain. Omit stops below `minExposureDuration` or above the v1 product cap. Never invent speeds the ISP cannot honor.
3. **Apply selection natively:**
   - Normal S-mode: `exposureMode = .continuousAutoExposure`, then **`activeMaxExposureDuration = CMTime`** for the selected speed. This is Apple's native shutter-priority ceiling — the same mechanism the system camera stack uses to cap exposure time while the ISP chooses ISO.
   - Read back **`exposureDuration`** (and ISO) for the HUD — live delivered values, not dial labels alone.
4. **Bright light:** delivered shutter may be **faster** than the dial (ceiling semantics). The readout shows what the device is actually doing.
5. **Beyond native max** (Bulb, stacked long): `setExposureModeCustom(duration:iso:completionHandler:)` and frame stacking **only** when the target exceeds `maxExposureDuration`. Still AVFoundation; clearly labeled in metadata where applicable.

**v1 product cap:** fast end of the dial stops at **1/2000 s** even when `minExposureDuration` allows 1/4000 or faster. Slow end runs down to the device's native minimum-duration floor (then Bulb/long if enabled). Example ladder when the format supports it: Bulb → 30 s → … → 1/1000 → **1/2000**. 1/4000 and faster are out of scope until the cap is raised.

**What we do not do in S-mode:** custom ISO hunting loops, timer-based fake shutter speeds, or exposure math that bypasses `activeMaxExposureDuration` / `setExposureModeCustom`.

### 6.2 Aperture-priority (A)

- The iPhone has a **fixed physical aperture** per lens. A-mode is exposure intent, not optical control.
- The dial sets f-stop intent (ƒ1.4 … ƒ16) via exposure target bias on `.autoExpose`. The system picks shutter and ISO.
- Physical aperture (e.g. ƒ1.78) is always shown separately in the readout.
- Optional simulated depth in A-mode preview when enabled — preview only, never written to archive. (Deferred if it requires a settings toggle; v1 keeps the capture surface self-contained.)

### 6.3 Mode switching

- Tap the mode letter (`S` or `A`) on the viewfinder to toggle instantly. Last-used mode persists across launches. Default first launch: **S**.

### 6.4 Metering and lock

- Tap the frame to set focus and exposure point.
- Long-press the frame to lock AE/AF. A lock glyph appears on the frame; `AE/AF` shows in the readout when locked.
- The exposure meter (tick needle + EV readout) appears while turning the dial and lingers briefly after.

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
│ [S]              1/250·ƒ1.78│   mode letter · live readout
│                  ISO 400    │
│                             │
│         LIVE FRAME          │   4:3 preview, real-time color look
│         (grid opt-in)       │
│                             │
│  ⟲                   1×   │   camera flip (selfie) · lens pill (rear)
├─────────────────────────────┤
│         +0.3  |▌|           │   exposure meter (when active)
│      ───|───|───|───        │   priority dial (S: shutter / A: ƒ-intent)
│  bolt    4:3  NEUTRAL       │   status strip (flash · grid · look — all tappable)
│  ▢          ◯       DNG/JPEG │   thumb · pill shutter · format label (tap toggles)
└─────────────────────────────┘
```

**Argentum mapping:** the center dial row is where Argentum shows `0.0 EV`; basic shows `1/250` or `ƒ4` instead. The row above the shutter is where Argentum shows `HCB` / `GW`; basic shows the active look name. The mode strip replaces Argentum's `NORMAL · STREET · DEPTH` row with display-only status (flash stub, `4:3`, look name) — **S/A lives in the top-left mode letter, not the strip.**

### Viewfinder overlays (on the 4:3 box only)

| Element | Behavior |
|---------|----------|
| Mode letter | Top-left. `S` or `A`. Tap toggles. Brass. |
| Live readout | Top-right. Shutter · physical aperture; ISO below (amber when maxed). |
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
| Exposure meter | Tick needle + EV value. Visible while dialing or when ISO maxed. |
| Priority dial | Argentum-scale tick ruler (horizontal portrait / vertical landscape). **Writes to native exposure APIs on session queue** (§4, §6). S-mode: attainable speeds through 1/2000 s. A-mode: ƒ-intent via bias. Double-tap resets to 1/250 / ƒ4. |
| Status strip | **Live controls** (Argentum row pattern, no gear): **flash** (tap) · **`4:3`** (tap toggles grid) · **look name** (tap opens filmstrip). No `NORMAL`/`STREET` modes. |
| Thumbnail | Last shot (Argentum bottom-left square). Tap opens Photos at saved asset. |
| Shutter | Cream pill rounded rect, brass border — same visual weight as Argentum's white pill. Tap fires. Long-press opens self-timer picker. Bulb on long-press when Bulb selected in S-mode. |
| Format label | Bottom-right (rear). Shows `DNG` or `JPEG`. Tap toggles format. Brass when DNG. In selfie mode: shows **`JPEG`** only; not tappable. |

---

## 8. Gestures and on-screen controls

**No settings page.** Every control is on the capture screen or a direct gesture below.

### On-screen (tap)

| Control | Location | Action |
|---------|----------|--------|
| Mode | Top-left `S` / `A` | Toggle shutter- vs aperture-priority |
| Format | Bottom-right `DNG` / `JPEG` | Toggle format (rear only; selfie = JPEG locked) |
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

### JPEG mode

- **JPEG** — processed still with the active look and color grade **baked in**. What you see in the viewfinder is what lands in Photos (Argentum WYSIWYG contract).
- Maximum quality JPEG settings at capture time. No separate RAW alongside.

### Format switching

- Toggle via the **`DNG` / `JPEG`** label on the capture screen (**rear only**).
- Format persists across sessions.
- In-flight captures complete in the format active when the shutter fired.
- **Selfie mode:** format is always **JPEG**. If the user flips to the front camera while DNG is selected, basic switches to JPEG automatically and restores DNG preference when returning to rear (if the user had DNG selected).

If Photos permission is denied, files write to an on-device sandbox with a persistent banner. Thumbnail and shot count still update.

---

## 12. Cameras — rear and selfie

### 12.1 Rear camera (default)

Runtime discovery on rear multi-camera hardware. Typical set on Pro models:

| Pill | Meaning |
|------|---------|
| 0.5× | Ultra-wide |
| Macro | Ultra-wide with near focus restriction |
| 1× | Wide (default) |
| 2× | Wide sensor crop |
| 4× | Telephoto |
| 8× | Telephoto crop (when hardware supports secondary native zoom) |

Swipe horizontal on the frame to change rear lens. Focal-length labels flash momentarily (device-native mm). Macro is manual only — optional assist banner when focus is near minimum on a non-macro lens.

Lens switching on the same physical sensor (1× ↔ 2×) uses zoom only — preview stays live. Cross-sensor swaps reconfigure the session on the **session queue** with minimal blackout (§4).

**DNG and JPEG** both available on rear when the SDK reports ProRAW support for the active rear format.

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
- Manual ISO/shutter mode (M)
- Program mode with EV compensation dial (P)
- EV compensation dial alongside the priority dial (Argentum's `0.0 EV` control is not ported — priority replaces it)
- S-mode shutter speeds faster than **1/2000 s** (v1 cap; may expand later)
- Argentum shooting modes: Street (full-screen shutter), Depth, Double exposure

**UI and product shape**
- Settings page, settings sheet, gear icon, preferences screen, or onboarding flow
- Level overlay, histogram, focus peaking, neutral-preview toggle, macro-assist banner config, A-mode depth settings UI
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
| Archive integrity | DNG bytes match SDK output (untouched); JPEG reflects active look (WYSIWYG) |
| S-mode dial | Ladder built from `activeFormat` exposure durations; `activeMaxExposureDuration` applied per stop; fast end at 1/2000 s; live `exposureDuration` in HUD |
| Grid toggle | Tap `4:3` on status strip toggles grid |
| No settings | Zero routes to a settings or preferences screen |
| Live looks | Neutral (color) default in viewfinder; Mono and Faded visible when selected |
| Format toggle | DNG ↔ JPEG on rear; selfie locked to JPEG |
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

---

*basic v1 · one screen · no settings · native stack · color · S/A · DNG or JPEG · iPhone Pro only*
