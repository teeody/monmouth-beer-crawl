# AGENTS.md — Pints & Pedals Bike Crawl site

Single-file static site (all CSS/HTML/JS inline). One real file: `index.html`,
plus `photos/` and `wave-rider.mp3` (music).

- Live: https://teeody.github.io/monmouth-beer-crawl/ (GitHub Pages)
- Remote: https://github.com/teeody/monmouth-beer-crawl.git
- Repository: /home/todd/monmouth-beer-crawl/

## Deploy workflow (ALWAYS)

1. Commit on `master`, push `master`.
2. `git checkout gh-pages` → `git merge master` → push `gh-pages`.
3. Wait for Pages build (`gh api repos/teeody/monmouth-beer-crawl/pages --jq .status` until `built`), then `curl` the live URL to sanity-check.
4. `git checkout master` to leave the working tree on `master`.

`gh-pages` is what GitHub Pages serves. Both branches must stay identical.

## Gotchas

- **Check `git branch --show-current` before every commit.** The share-menu and
  Instagram commits were once accidentally made on `gh-pages`; recovered by
  fast-forwarding `master` from `gh-pages`.
- **There are TWO identical lines** in the inline script near the end:
  `document.getElementById('interestCount').textContent = ...` (one inside the
  signupForm submit handler, one just before `</script>`). The submit handler
  one must stay inside its callback — anchoring an edit on the bare line can
  grab the wrong occurrence and scope a whole block to the wrong function.
- `index.html` is ~1500 lines; **use `rg -n` to locate sections** — line
  numbers shift after every edit.
- Keep edits small; huge single-payload edits risk "Unterminated string" /
  JSON truncation errors.
- Two infinite SMIL `<animateMotion>` loops (fish on the ocean, bike on the
  route) run on pageload — this makes `chromium --dump-dom` snapshots
  unreliable. Verify JS with headless Chromium over CDP
  (`Runtime.evaluate`) instead.

## Verify JS locally

Inline script is the only `<script>` block (starts ~line 1253, ends `</script>`):

```
sed -n '/<script>/,/<\/script>/p' index.html | sed '1d;$d' > /tmp/site.js && node --check /tmp/site.js
```

Key JS functions: `rideDates`, `tickCountdown` (countdown), `downloadICS`
(.ics file), `goStop(n)` (clickable map stops), `pokeFish`/`fishJump` (fish),
`bikeRain`/`confettiBurst` (easter eggs), `openShare`/`closeShare`/`shareVia`/
`copyShareLink` (share menu), `toggleMusic` (🎵/⏸).

## Features map (JS functions)

- Hero countdown → rideDates(), tickCountdown() — dates parsed from the
  signup `ride_date` select (29 Sundays Apr 18–Oct 31 2027, 10:00 local).
- Clickable map stops → .map-stop[data-stop=N] ↔ .stop-card#stop-N, goStop(n).
- Fish jump → .fish-scene onclick fishJump(event); type "fish" triggers it too.
- Bike on route → .route-bike-wrap <animateMotion> on the dashed route path.
- Easter eggs → typing "pints" rains 🚴; 5 quick clicks on the share button
  fire confetti.
- Music → wave-rider.mp3 ("Wave Rider" by Shane Ivers, silvermansound.com,
  CC BY 4.0), bgm.volume = 0.3.
- Music credit line in the footer: Music: "Wave Rider" by Shane Ivers
  (silvermansound.com) — CC BY 4.0.

## Design tokens (index.html CSS vars)

- Font: Permanent Marker (Google Fonts), site-wide.
- Colors: navy #1A5276 / ink #1A252F, sky gold #5DADE2 + #85C1E9, teal #17A2B8
  (+ #20C997), coral #FF6B6B, amber #FFB347 / #E2A33C, cream #F0F4F8 /
  #D6E4F0 / #B8CCE0.

## Notes / history

Session work log & gotchas for prior sessions live in:
`~/backups/monmouth-beer-crawl-session-notes-*.txt` (also good to re-read for
context; this file is the canonical agent-facing summary).