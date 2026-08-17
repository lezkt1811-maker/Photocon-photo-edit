# Dev Lab — Photo Theming & 300 DPI Upscale

A single-page, no-install photo tool: drop in a photo, pick a theme, export
it upscaled with a real 300 DPI tag written into the file. Everything runs
in the browser via Canvas — no server, no upload, no API costs.

**[Live demo →](#)** *(update once GitHub Pages is on, see below)*

## What it actually does (read this before you rely on it for print)

**Themes** are real color-grading + light/texture processing — CSS filter
adjustments (contrast, saturation, hue), gradient overlays with proper blend
modes, vignettes, film grain, bloom, and scanlines, layered per theme. It's
the same category of technique a photo editor uses, applied automatically.
It is **not** a generative AI re-draw — it can't invent objects or repaint
a photo into a wholly different scene. That would require a real image
generation model behind an API.

**Upscaling** increases pixel dimensions using a stepped high-quality
resample (doubling at each step rather than one big stretch, which keeps
edges cleaner than a single-pass resize). It does **not** invent detail the
original photo didn't capture — true detail-adding upscaling needs a
super-resolution ML model. If you need that, this tool is a good first
pass before running the output through something like Real-ESRGAN.

**300 DPI** is written as real embedded metadata, not just a claim in the
filename:
- PNG: a proper `pHYs` chunk (pixels-per-meter, computed from 300 DPI) is
  inserted after the `IHDR` chunk, with a correct CRC32.
- JPEG: the JFIF `APP0` header's density fields are patched to 300 DPI.

Print-on-demand sites (Redbubble, etc.) and most design software read this
correctly.

## Included themes

| Theme | Look |
|---|---|
| Cyberpunk | Magenta/cyan split gradient, punched contrast, night grain |
| Cosmic Rainbow Glow | Soft conic rainbow wash + bloom halo |
| Dark Lilith Dreamscape | Blood-violet shadows, low brightness, heavy grain |
| Golden Hour Film | Warm sepia-lifted highlights, soft film grain |
| Vaporwave Dusk | Pink/blue duotone with retro scanlines |
| Noir Silver | High-contrast monochrome, deep vignette, heavy grain |

## Adding your own theme

Everything lives in one array in `index.html` — `const THEMES = [...]`.
Copy an existing entry and adjust:

```js
{
  id: 'my-theme',
  name: 'My Theme',
  desc: 'Short description shown on the picker card.',
  swatch: ['#111111', '#ff0000', '#ffffff'], // 3 colors for the thumbnail
  filter: 'contrast(1.1) saturate(1.3) brightness(1.0)', // any CSS filter string
  overlay: { type:'linear', angle:135, stops:[[0,'#ff0000'],[1,'transparent']], blend:'screen', alpha:0.3 },
  effects: [
    { type:'grain', intensity:0.08 },
    { type:'vignette', strength:0.4, color:'#000000' }
  ]
}
```

Overlay `type` can be `linear`, `radial`, or `conic`. Effect `type` can be
`grain`, `vignette`, `rim`, `scanlines`, or `bloom` — see the functions near
the bottom of `index.html` if you want to add a new effect type entirely.

## Running it locally

No build step. Just open `index.html` in a browser — this one works fine
directly via `file://` since it doesn't fetch external data files.

## Deploying to GitHub Pages

1. Push this folder to a GitHub repo.
2. **Settings → Pages → Source**: deploy from branch `main`, folder `/ (root)`.
3. Live at `https://<username>.github.io/<repo>/`.

## Structure

```
photo-dev-lab/
├── index.html   # everything — UI, theme presets, canvas pipeline, DPI writer
└── README.md
```

## Stack

Vanilla HTML/CSS/JS, Canvas API, zero dependencies, zero external requests
(other than Google Fonts for the UI typefaces).
