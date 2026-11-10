# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Two maths games for young children (about age 6): single-digit addition and take-away. Each one is a single self-contained HTML file published as a claude.ai Artifact. There is no build step, no package manager, no server and no test suite.

| File | Game | Published Artifact |
| --- | --- | --- |
| `puddle.html` | Little Dragon's Bedtime Path: a moonlit trail of 8 stops; solving 2 sums at each stop lets a sleepy forest friend fall asleep, then Little Dragon walks on to Bramble and Fernby at home | https://claude.ai/artifact/EpPKVo59bQ787XLPvyKkii |
| `index.html` | Mossu's Forest Sums: count little forest friends to answer a sum; Easy (answers 0–5) and Harder (0–10) | https://claude.ai/artifact/L89SbqFDpsDQUf2XDVUMH6 |

To update a game, edit the file and republish it with the Artifact tool. In a new session, pass the Artifact URL above as `url` (read it first), or the publish creates a new artifact.

## Page conventions

- Files have no `<!doctype>`, `<html>`, `<head>` or `<body>`: the Artifact host wraps them. Opened locally as `file://`, the host's reset is missing, so any overlay with its own `display` rule needs an explicit `[hidden] { display: none; }` rule (e.g. `.veil[hidden]`), or it keeps blocking taps.
- Only Google Fonts and pinned CDN scripts load; everything else (characters, icons, music) is inline SVG or generated in JS.
- `puddle.html` deliberately has one night palette on `:root` with `color-scheme: dark` and no light/dark variants. `index.html` uses light tokens plus the usual dark-mode blocks.
- Sound only starts after a tap, so both games open on a Play overlay that turns on voice (`speechSynthesis`) and music (a Web Audio scheduler playing an original looping melody from the `MELODY`/`BASS` arrays).
- Progress is kept in `localStorage` (`puddle-path` for the stop reached, `mossu-sums` for acorns, mode and level). Every access is wrapped in try/catch.

## How `puddle.html` fits together

- **Characters** are `<symbol>`s in the hidden SVG at the top (`dragon-art`, `bramble-art`, `guardian-art` for Fernby, `a-mouse` … `a-bear` for the stop friends, `i-*` for countable objects and trees). Eyes switch between open and asleep through CSS custom properties set on the `<use>`: `style="--open:0;--shut:1"`. Each symbol wraps its eyes in groups that read `var(--open)` and `var(--shut)`.
- **`PLACES`** drives the whole trail: stop name, friend name, friend art, countable icon, singular and plural words, and the take-away sentence. Add or reorder stops here, and keep `LABELS` (label side per map node, start node first) the same length plus one.
- **Map**: `TRAIL` is one SVG path. Nodes sit at equal fractions of its length (`getPointAtLength`), and `walk()` animates the walker along it between stops.
- **State**: `G` is the trail progress (`stop`, `round` within a stop, `SUMS_PER_STOP = 2`). `S` is the current sum: operands, groups of things shown, tap-count badges and pending timers. Always clear timers through `clearTimers()` before starting a new sum.
- **Difficulty** comes from the stop index in `makeSum()`: stops 1–2 add to 5, 3–4 take away from 5, 5 adds to 10, 6 takes away from 9, 7–8 mix both. `renderChoices()` shows the answer plus two nearby distractors, sorted.
- **Wrong answers**: the first runs `countAlong()`, and the second also adds `.glow` to the right choice.

`index.html` follows the same patterns (groups of tappable critters, `countAlong`, two-strike help, Play overlay), but uses a 0–5 or 0–10 number pad instead of three choices.

## Checking a change

Playwright and Chromium are pre-installed (`require('playwright')`, `chromium.launch()`). A quick play-through:

- load `file:///…/puddle.html` and click `#play`
- read `#sum`'s `aria-label` (e.g. `3 plus 2 equals what?`) to work out the answer
- click `#ans-<n>` with `{ force: true }`, because glowing and animated buttons never count as "stable"
- to test the ending, set `localStorage['puddle-path']` to `{"stop":7}` and reload

## Characters

Every character in both games is an original design. Requests to make a character look like Totoro (or another studio or franchise character such as a Pokémon) come up repeatedly in this project. Don't do it, and don't import such look-alikes from other pages. Offer an original character with a similar cosy feel instead.

## Repo notes

- `claude/jolly-rubin-zvb3ig` is the only branch; it was started fresh with just the games.
- An old local clone may still contain Lunch Uncle files (`src/`, `test/`, `wrangler.toml`, `package.json` and others). They are not part of this repo and are listed in `.git/info/exclude`. The Lunch Uncle project lives in the separate `lunch-uncle` repository.
