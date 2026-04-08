# Little Paper Things — Website Project Brief (AI.md)

## What This Is

A website for **Little Paper Things (LPT)**, a subscription snail mail surprise gift envelope business. Customers subscribe to receive hand-curated paper treasures (stickers, bookmarks, cards, small paper goods) in a surprise envelope delivered by post each month.

## Tech Stack

- **Static site generator:** Hugo
- **Hosting:** Netlify
- **CMS:** Sveltia CMS (drop-in Decap CMS replacement, same config format)
- **Auth:** Netlify Identity
- **Repo:** GitHub (Netlify deploys from the repo)

Sveltia CMS is critical — the business owners are non-technical and need a friendly editing interface. Every piece of editable content should be backed by CMS-manageable front matter or data files.

## Design Direction

**Aesthetic:** Elegant paper-letter stationery — warm, handcrafted, analog. The site should feel like receiving a beautiful letter, not like a tech product. Paper textures are used as **subtle accents** (not the entire design language). The tone is warm, personal, and a little bit magical.

**NOT:** Skeuomorphic, kitschy, scrapbooky, or over-textured. The paper aesthetic should be restrained and modern — think Maurèle (maurele.com) meets a hand-lettered love note.

---

## Colour Palette

All colours defined as CSS custom properties in `:root`.

### Base tones (paper)
| Name | Hex | Usage |
|------|-----|-------|
| `--paper-white` | `#faf8f4` | Main page background (cream paper) |
| `--paper-warm` | `#f0ebe1` | Card/section backgrounds (heavier stock feel) |
| `--paper-edge` | `#e8dfd0` | Borders, dividers, texture overlay tone |

### Ink tones (text)
| Name | Hex | Usage |
|------|-----|-------|
| `--ink-dark` | `#2c2420` | Primary text, headings (warm near-black) |
| `--ink-light` | `#5c4f42` | Secondary text, captions, metadata |

### Accent colours
| Name | Hex | Usage |
|------|-----|-------|
| `--wax-red` | `#a03c2f` | Primary accent — CTAs, links on hover, logo pinwheel highlight |
| `--twine-gold` | `#c4993b` | Secondary accent — borders, decorative rules, stamp/seal elements |
| `--sage-green` | `#6b7f5e` | Tertiary — envelope elements, subtle highlights (use sparingly) |

---

## Typography

All fonts from Google Fonts. Load via `<link>` in the head or self-host.

| Role | Font | Weight(s) | Usage |
|------|------|-----------|-------|
| Headings / hero | **Cormorant Garamond** | 400, 500, 600, 400i | Page titles, section headings, hero text |
| Handwriting accent | **Caveat** | 400, 500 | Taglines, pull-quotes, short labels like "with love", signature flourishes. **Never for body text.** Min 24px. |
| Body text | **Source Serif 4** | 400, 600, 400i | All paragraph text, article content |
| UI elements | **Source Sans 3** | 400, 600 | Nav links, buttons, form labels, small caps, metadata |

### Font stack fallbacks
```css
--font-heading: 'Cormorant Garamond', Georgia, 'Times New Roman', serif;
--font-accent: 'Caveat', cursive;
--font-body: 'Source Serif 4', Georgia, serif;
--font-ui: 'Source Sans 3', -apple-system, sans-serif;
```

---

## Paper Texture (CSS-only, no images)

Two layers applied to the body/main wrapper:

### Layer 1 — Base gradient
Very subtle radial gradient, warmer in the centre, cooler at edges. Almost invisible but prevents flat-digital-white feeling.
```css
background: radial-gradient(ellipse at 50% 30%, #faf8f4, #f5f0e8);
```

### Layer 2 — Noise grain overlay
SVG `feTurbulence` noise as an inline data URI, applied to a `::before` pseudo-element with `mix-blend-mode: multiply` and very low opacity (0.03–0.05). No image file needed.
```css
body::before {
  content: '';
  position: fixed;
  inset: 0;
  background-image: url("data:image/svg+xml,%3Csvg width='100' height='100' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.8' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100' height='100' filter='url(%23n)' opacity='0.04'/%3E%3C/svg%3E");
  pointer-events: none;
  z-index: 9999;
}
```

### Cards and sections
Cards use `--paper-warm` (#f0ebe1) with a subtle `box-shadow` to feel like a card laid on a desk. The registration/subscription form should use slightly heavier texture (opacity ~0.08) to feel like thicker card stock.

---

## Logo

The logo is in `static/images/lptlogogrouped.svg`. It is a hand-drawn SVG created in Inkscape containing:

### SVG structure
```
<svg viewBox="0 0 166.69 110.75">
  <g transform="translate(-21.657 -165.43)">
    ├── #logo-text          (filled path — "little paper things" in Calligraphy Brillian)
    └── #pinwheel           (group)
        ├── #stick           (diagonal line connecting text to pinwheel)
        └── #pinwheel-blades (group — 5 blade shapes + crease lines)
            └── #pinwheel-center (ellipse — the centre dot)
  </g>
</svg>
```

### Pinwheel centre coordinates
For CSS `transform-origin` on `#pinwheel-blades` to spin around the centre dot:
- Computed from the rotated circle element: approximately `146.4px, 191.2px` in the SVG's internal coordinate system
- In viewBox-relative coordinates (after the parent translate): approximately `124.7px, 25.8px`
- **These may need fine-tuning** — if the spin wobbles, nudge by a few pixels until it rotates cleanly around the dot

The logo text is a single filled path (not a stroke font), so `stroke-dashoffset` write-on animation does NOT work directly on it. Use clip-path wipe or SVG mask techniques.

---

## Logo Loading Animation

The logo IS the site loader. Three-phase animation sequence:

### Phase 1: Pinwheel spins (0s–2.4s)
- `#pinwheel-blades` rotates continuously: `animation: spin 1.2s linear infinite`
- `#stick` fades in: `opacity: 0 → 1` over 0.4s with 0.3s delay

### Phase 2: Text reveals (0.8s–2.3s)
- `#logo-text` wipes left-to-right using `clip-path: inset(0 100% 0 0)` → `inset(0 0% 0 0)`
- 1.5s duration, `cubic-bezier(0.25, 0.1, 0.25, 1)`, 0.8s delay

### Phase 3: Decelerate + dock (2.4s–3.3s)
- Pinwheel switches from infinite spin to a single 540° rotation with `cubic-bezier(0.2, 0, 0, 1)` ease-out
- Entire logo container scales down and translates to its final nav corner position
- Loader overlay fades out, page content fades in (0.5s)

### Skip on revisit
Store a flag in `sessionStorage` — on subsequent page loads within the same session, show the logo already docked in the corner, skip the animation.

### Reference
The file `reference/lpt-animation-prototype.html` is a standalone working prototype of this animation with the actual SVG. The pinwheel transform-origin may need adjustment.

---

## Envelope Animation for Subscription Form

When the user scrolls to the subscription section (or clicks a CTA), an envelope opens to reveal the signup form:

1. Static envelope is visible as a decorative element
2. On trigger: envelope flap opens via `rotateX` with `transform-origin: top`
3. Form slides up out of the envelope body via `translateY`
4. Form is styled as a paper card (lighter background, soft shadow)

### Reference CodePens
- codepen.io/MrBlank/pen/JjXxovL — envelope open/close with letter
- codepen.io/lenasta92579651/pen/NWdeYYb — pure CSS hover envelope
- codepen.io/amiriftikhar/pen/KKbPdww — clean open-on-hover with shadow

The common technique: envelope shapes from CSS `border` triangles, flap uses `transform-origin: top` + `rotateX`, letter slides up with `translateY`.

---

## Site Pages & Content Structure

### Pages needed
1. **Home** — hero, value prop, how-it-works, past envelopes preview, CTA
2. **How It Works** — step-by-step explanation
3. **Past Envelopes** — gallery/archive of previous months (CMS-editable)
4. **Gift a Subscription** — gifting flow / info
5. **About** — the story, the people
6. **Subscribe / Pricing** — plans and signup (envelope animation lives here)

### CMS-editable content
All page content should live in Hugo front matter or `data/` YAML files so the owners can edit via Sveltia CMS. This includes:
- Hero text and tagline
- How-it-works steps
- Past envelope entries (title, description, image)
- Pricing/plan details
- About page copy

---

## Hugo Directory Structure

```
lpt-site/
├── AI.md                    # This file
├── hugo.toml                    # Hugo config
├── content/
│   ├── _index.md                # Home page
│   ├── how-it-works.md
│   ├── past-envelopes/
│   │   └── _index.md
│   ├── gift.md
│   ├── about.md
│   └── subscribe.md
├── layouts/
│   ├── _default/
│   │   ├── baseof.html          # Base template with loader
│   │   ├── single.html
│   │   └── list.html
│   ├── index.html               # Home page template
│   └── partials/
│       ├── head.html            # Font loading, meta, CSS
│       ├── nav.html
│       ├── footer.html
│       ├── logo-loader.html     # The animated logo loader
│       ├── logo-static.html     # Static logo for nav (after dock)
│       └── envelope-form.html   # Envelope reveal signup form
├── static/
│   ├── images/
│   │   └── lptlogogrouped.svg   # The logo
│   └── admin/
│       └── index.html           # Sveltia CMS admin page
├── assets/
│   └── css/
│       └── main.css             # Main stylesheet
├── data/                        # CMS-editable data files
├── reference/
│   └── lpt-animation-prototype.html  # Animation prototype (not deployed)
├── static/admin/
│   ├── index.html               # Sveltia CMS entry point
│   └── config.yml               # Sveltia/Decap CMS config
├── netlify.toml                 # Netlify build config
└── .gitignore
```

---

## Sveltia CMS Setup

### Admin page (`static/admin/index.html`)
```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Content Manager</title>
  <link href="https://unpkg.com/@sveltia/cms/dist/sveltia-cms.css" rel="stylesheet" />
</head>
<body>
  <script src="https://unpkg.com/@sveltia/cms/dist/sveltia-cms.js"></script>
</body>
</html>
```

### CMS config (`static/admin/config.yml`)
Uses Netlify Identity for auth, Git Gateway as backend. Collections for each content type.

### Netlify Identity
Enable Netlify Identity in the Netlify dashboard. Add the Identity widget script to the site head and an init snippet. Invite the business owners as Identity users.

---

## Netlify Config (`netlify.toml`)
```toml
[build]
  command = "hugo"
  publish = "public"

[build.environment]
  HUGO_VERSION = "0.147.1"

[context.production.environment]
  HUGO_ENV = "production"
```

---

## Key Design References

### Paper/texture CSS
- freefrontend.com/css-paper-effects/ — 34 CSS paper effects with live demos
- transparenttextures.com — preview paper textures with any colour
- tutorialpedia.org guide to pure CSS old paper texture (radial gradients + noise)
- Erik Ritter's texture overlay technique (mix-blend-mode: multiply + pointer-events: none)

### Envelope animations
- codepen.io/MrBlank/pen/JjXxovL — open/close with hearts (adapt for form reveal)
- codepen.io/amiriftikhar/pen/KKbPdww — clean open-on-hover
- codepen.io/lenasta92579651/pen/NWdeYYb — pure CSS open/close on hover

### SVG handwriting animation (for future logo upgrade)
- css-tricks.com article on handwriting animation with irregular SVG strokes (masking technique)
- codepen.io/marinamcpeak/pen/XYNdvd — per-letter clip masks with staggered animation-delay
- svgator.com/blog/how-to-create-a-handwriting-animation/ — full walkthrough

### Pinwheel loader
- codepen.io/lindarenee/pen/KKBaOX — pure CSS pinwheel spinner

### Site inspiration
- maurele.com — tasteful stationery brand, clean modern-classic balance
- thepostmansknock.com — calligraphy/handwriting blog with warm crafty aesthetic
- makingmeadows.co.uk — UK eco-stationery shop, warm gentle tone

---

## Important Notes

- Hugo builds are fast — the logo loader will typically only display for 200–500ms on cached visits. Enforce a minimum display time of ~800ms so the animation registers visually.
- The logo SVG is ~50KB. For production, consider running it through SVGO to strip unnecessary precision and metadata.
- The pinwheel `transform-origin` coordinates may need fine-tuning. If the spin wobbles, adjust by a few pixels.
- The text path in the logo uses a font called "Calligraphy Brillian" — this is baked into the SVG as outlines, so no font dependency.
- All design tokens (colours, fonts, spacing) should be CSS custom properties for easy theming.
