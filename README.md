# basic — marketing site source

A dark, editorial one-page marketing site for *basic*, a priority-mode (S/A) camera app for iPhone.

## What's in here

```
basic-site.dc.html        Main page (source). All layout + copy + inline styles.
CaptureMock.dc.html       The reusable iPhone capture-screen mock used throughout the page.
support.js                Runtime that powers the two .dc.html files.
image-slot.js             Drag-to-drop image placeholder web component (used for the app icon).
basic-site.standalone.html Single-file build with the runtime inlined (still reads from images/).
images/                   All photography used on the page.
```

## Images and where they appear

- `hero-car.jpg` — hero capture mock (street frame)
- `look-boat.jpg` — the three "Looks" cards (Neutral / Faded / Mono)
- `selfie.jpg` — the "Flip it around" front-camera mock
- `app-marine.jpg` — "One screen. The whole app." — left tile (Neutral / DNG)
- `app-bar.jpg` — "One screen…" — middle tile (Mono)
- `app-monk.jpg` — "One screen…" — right tile (Faded / JPEG)

## Viewing it

Open `basic-site.dc.html` (or `basic-site.standalone.html`) from inside this folder with any
static server (e.g. `npx serve .`) so the relative `images/` and script paths resolve.

## Notes for conversion

- Everything is styled with **inline styles** — there is no external CSS file. Each capture-mock
  instance is configured entirely through HTML attributes (`mode`, `shutter`, `ap`, `iso`,
  `look`, `lens`, `format`, `pos`, `size`, `photo`).
- Fonts: **Spectral** (Google Fonts, headings/serif) + system **SF Pro / -apple-system** (UI)
  and **SF Mono** (numerals). Accent color is brass `#C9A35B`; background is near-black `#000`.
- The capture mock's look filters (Neutral / Mono / Faded) are CSS `filter` values defined in
  `CaptureMock.dc.html`.
