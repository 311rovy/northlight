# Northlight — Agency Platform

An **editorial-brutalist**, WebGL/motion-forward marketing site and client platform for a fictional
creative agency & production house ("Northlight") — colossal grotesk on a hard grid, one hot flare
accent, and pervasive-but-cohesive motion. Built as a showcase of high-end design/motion craft on
top of Next.js. See [PLAN.md](PLAN.md) for the full page map and status.

The marketing site (landing, work grid, case studies) leans on:
- A full-viewport **WebGL halftone field** (raw GLSL, R3F) behind the hero — a flowing FBM dot
  field that ripples under the pointer, with a static dot-grid fallback via a capability gate.
- **GSAP ScrollTrigger** section reveals, two deliberate pinned/scrubbed scroll moments (Selected
  Work crossfade + case study Approach steps + scrubbed video), and a scroll-progress "story spine".
- **RGB-shift glitch** image reveals (SVG channel filters), infinite CSS **marquees**, a
  blend-difference **custom cursor**, **magnetic** buttons, and a brutalist **route-wipe** transition.
- Momentum **drag carousels** (GSAP Draggable + InertiaPlugin) for the client logo row and gallery.

All motion respects `prefers-reduced-motion` and gates WebGL off on touch / no-WebGL devices. The
client platform area (`/login`, `/dashboard`, `/projects`, `/briefs`) shares the brutalist system in
an app-like layout, running on mock, in-memory data — no backend.

## Getting started

Install dependencies:

```bash
npm install
```

Run the dev server:

```bash
npm run dev
```

Then open [http://localhost:3000](http://localhost:3000) (or whatever port is printed — the dev
server auto-picks a free one if 3000 is busy).

## Testing

Run the Playwright end-to-end suite (covers the landing hero, case study navigation, scroll/hover
behavior, the dashboard project list, and the brief-creation flow):

```bash
npm run test:e2e
```

Or with the interactive UI runner:

```bash
npm run test:e2e:ui
```

The suite boots its own dev server on port 3100 (see `playwright.config.ts`) so it doesn't collide
with a `next dev` instance you may already have running on 3000.

## Deploying to Vercel

The whole site is statically generated (no backend), so it deploys to Vercel with zero config.

**Set these environment variables in the Vercel project** (Project → Settings → Environment
Variables), for the Production (and Preview) environments — they are NOT committed:

| Variable | Value |
| --- | --- |
| `UNSPLASH_ACCESS_KEY` | your Unsplash access key (see your local `.env.local`) |
| `NEXT_PUBLIC_SPLINE_SCENE` | your Spline scene URL (see your local `.env.local`) |

Then either:

- **GitHub → Vercel (recommended):** push this repo to GitHub, then "Add New Project" on
  [vercel.com/new](https://vercel.com/new), import it, add the env vars above, and Deploy.
- **Vercel CLI:** `npm i -g vercel`, then `vercel` (follow the login prompt) and `vercel --prod`.
  Add the env vars with `vercel env add UNSPLASH_ACCESS_KEY` / `vercel env add NEXT_PUBLIC_SPLINE_SCENE`.

Vercel auto-detects Next.js; the build command is `next build` and the output is handled for you.
Images come from Unsplash at build time — on the free demo key (50 req/hr) a very rapid series of
redeploys could briefly rate-limit a few images into their gradient fallback; they refresh on the
next build.

## Other scripts

```bash
npm run build   # production build
npm run start   # run the production build
npm run lint    # eslint
```

## Project structure

- `src/app/(marketing)/` — landing, `/work`, `/work/[slug]` case studies. Shares `SiteHeader` /
  `SiteFooter`, Lenis smooth scroll, and the custom cursor.
- `src/app/(platform)/` — `/login` (standalone) and, under the nested `(app)` group,
  `/dashboard`, `/projects`, `/projects/[id]`, `/briefs`, `/briefs/new` — wrapped in `AppShell`.
- `src/components/motion/` — shared motion primitives (`FadeIn`, `SlideUp`, `ParallaxImage`,
  `DragCarousel`, `Marquee`, `Magnetic`, `PageTransition`, smooth-scroll and cursor providers).
- `src/components/three/` — `HalftoneField` (the WebGL hero shader) plus the earlier `HeroScene`,
  and `SceneGate`, which gates WebGL behind reduced-motion / mobile / WebGL-support checks.
- `src/components/MediaFrame.tsx` / `GlitchFilters.tsx` — image component with optional RGB-shift
  glitch, and the shared SVG colour-channel filters it references.
- `src/components/marketing/`, `src/components/case-study/`, `src/components/platform/` —
  section-level components for each area.
- `src/lib/motion.ts` — centralized easing/duration tokens used by every GSAP timeline.
- `src/lib/case-studies.ts`, `src/data/projects.ts`, `src/data/briefs.ts` — mock content/data.
- `tests/e2e/` — Playwright specs.

## A note on imagery

There's no real photography or video, so imagery is sourced live from the Unsplash API
(`src/lib/unsplash.ts`), keyed by a search query per project/gallery slot in
`src/lib/case-studies.ts`. Copy `.env.local.example` to `.env.local` and set
`UNSPLASH_ACCESS_KEY` (free key at [unsplash.com/developers](https://unsplash.com/developers)) to
enable it. Images come from Unsplash's **search** endpoint ordered by relevance (top hit), served
at ~1680px via imgix params. Without a key — or if a fetch fails — `MediaFrame` falls back
automatically to a generative gradient + grain placeholder, so the site always renders either way.

## Adding a Spline 3D scene

The hero uses a WebGL halftone shader by default, but you can drop in a **Spline** scene instead —
the integration is already wired (`src/components/three/SplineEmbed.tsx`, gated by `SceneGate`).

1. In [spline.design](https://spline.design), open your scene (or remix a community one).
2. Top-right **Export → Code / React**, choose **Public URL**, and copy the URL — it ends in
   `.splinecode` (e.g. `https://prod.spline.design/XXXXXXXX/scene.splinecode`).
3. In `.env.local`, set `NEXT_PUBLIC_SPLINE_SCENE=` to that URL and restart `npm run dev`.

The hero swaps the halftone for your scene automatically. Leave the var blank to keep the halftone.
Tip: scenes with a transparent background sit best behind the headline; keep them visually calm so
the type stays the focal point.
