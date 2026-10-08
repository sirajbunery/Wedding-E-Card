# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page, no-build wedding invitation e-card for the joint wedding of Waqar Ahmad & Siraj Ahmad Khan (25 October 2026, Kingargali, District Buner, KPK, Pakistan). The entire site is one self-contained file: `index.html`.

## Development workflow

There is no build system, package manager, or test suite — it's plain HTML/CSS/JS in one file loaded directly by the browser.

- **Run it**: open `index.html` directly in a browser, or serve the directory with any static file server (e.g. `npx serve .`) for correct relative-path/audio-autoplay behavior.
- **No lint/build/test commands exist.** Verify changes by opening the page in a browser and checking the relevant interaction manually (envelope tap, language toggle, theme toggle, countdown, calendar download, music).
- Deployed via Vercel from this repo (static site, zero config needed).

## File layout

- `index.html` — everything: `<head>` meta/fonts, one big `<style>` block, the full page markup, one `<script>` block at the end. Line ranges (approximate, re-check after edits): styles `14–590`, envelope/stage markup `594–619`, hero section `621–685`, details/contacts section `687–748`, floating toggle buttons + audio element `750–767`, script `769–1021`.
- `assets/ambient-piano.mp3` — background music played once the envelope opens.
- `README.md` — one-line project description only.
- `findings.md`, `progress.md`, `task_plan.md` — scratch planning notes from prior sessions (not part of the shipped site; safe to ignore unless continuing that specific planning thread).

## Architecture of `index.html`

**Flow:** `#stage` (envelope intro, `z-index:50`, covers viewport) → tap/click/Enter triggers `openInvite()` → envelope flap animation + petal burst → stage fades out, `#hero` becomes visible and scroll unlocks → auto-scroll to `#details` fires after 10s unless the visitor has already scrolled.

**Bilingual EN/Urdu content** is data-driven, not duplicated markup: every translatable element carries both `data-en` and `data-ur` attributes; `applyLang(lang)` (script section, near top) walks all `[data-en]` elements and swaps `textContent`, flips `dir`/`lang` on `<html>`, and toggles a `urdu` class on `<body>` that switches the font stack to Noto Nastaliq Urdu. When adding new user-facing text, always add both attributes and matching initial English text content — don't hardcode English-only strings.

**Countdown & calendar stay in sync by sharing one source of truth**: `WEDDING_START` / `WEDDING_END` constants near the top of the script. The live countdown, the generated `.ics` file (built inline as a VCALENDAR string blob, not an external library), and the displayed event time all derive from these — update them together if the event date/time changes, not the display strings in the markup.

**Day/night theme** auto-detects from local clock hour (6am–6pm = light) but a manual toggle click always wins and persists via `localStorage` (`wedding-theme` key). The toggle uses the View Transitions API for a radial reveal animation from the button's position, with a plain instant swap fallback for unsupported browsers or `prefers-reduced-motion`. See the inline comment at the `transition.ready.then(...)` double-`requestAnimationFrame` — it exists to dodge a real timing race where the browser hasn't painted the 0px clip-path yet; don't simplify it away without understanding that race.

**Motion is centrally gated** by `prefersReducedMotion` (checked once near the top of the script from `matchMedia('(prefers-reduced-motion: reduce)')`) — ambient petal spawning, scroll parallax on the hero foliage SVGs, and the view-transition theme swap all branch on this flag. New animated features should check it the same way rather than relying solely on the CSS `@media (prefers-reduced-motion: reduce)` blanket rule.

**Scroll-triggered reveals** use an `IntersectionObserver` over all `.reveal` elements, adding `.in-view` with a small staggered delay per batch; falls back to revealing everything immediately if `IntersectionObserver` is unsupported.

**Background music** autoplays on envelope open via `setMusicPlaying(true)` (muted/blocked failures are caught and logged, not surfaced as errors) and can be toggled independently via the floating music button; state is reflected with a `playing` class and `aria-pressed`.

**Styling conventions**: CSS custom properties in `:root` define the palette (`--ivory`, `--bottle`, `--gold`, `--wine`, etc.) and a shared `--ease` cubic-bezier — reuse these rather than introducing new literal colors/timings. Sequenced entrance animations on hero content use a `seq-N` class convention (`seq-1` through `seq-9`) for staggered delays.
