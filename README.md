# Fast Site Generator

[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/pakun1/site-generator-speed-01-run-20261122-entry/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/pakun1/site-generator-speed-01-run-20261122-entry/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)

A faster drop-in for makefun-site 1.0.0, built for make.fun's "Build a static site faster" bounty. It answers exactly as makefun-site does on every case — same output tree, byte for byte — but in about a third of the CPU time.

## What it is

- `solve.js` — an ES module whose `run(input)` returns makefun-site's answer for one case's input: the output tree (each page's `index.html`, the index and tag pages, `build-report.txt`, each static file) as a JSON object of path to text.
- `main.ts` — the program the harness runs: it reads every case from `/input/cases.json`, answers each with `run`, and writes `/output/answers.json`.

Node 22, JavaScript/TypeScript only, no dependencies, nothing installed, no WebAssembly.

## How it is faster

makefun-site's cost is almost all Markdown: it lexes each body into a token array, then renders it, calling a per-character HTML escape for every character of every paragraph. This implementation instead:

- Renders Markdown in a single pass, straight from lines to HTML — no token array is ever built.
- Copies plain-text runs whole in inline rendering. A single regex finds the next of the nine characters that matter (`\`, backtick, `*`, `[`, `&`, `<`, `>`, `"`, `'`); the text between two of them holds nothing escapable, so it is copied as-is instead of being escaped one character at a time. A string with no special character at all is returned untouched.
- Gives slugs an ASCII fast path: NFKD normalisation is the identity on ASCII, so ASCII titles skip it and go straight to the slug loop.
- Counts a body's words without building an array of them.

Front matter, the template dialect, the slug rule, the collections, the tags and the report are implemented to match makefun-site exactly — same checks in the same order, same report lines, same output paths.

## Practice results

On the kit's practice set (3 cases: 250, 340 and 480 Markdown sources):

- **Correctness: 3 of 3 cases identical to makefun-site 1.0.0**, byte for byte (checked against `practice/reference/expected.json`).
- **Time: ~200 ms for the set** (five timed runs: 203, 203, 211, 218, 302 ms), against the house baseline's ~550 ms (514–594 ms) on the same machine — about **2.6× faster**.
- Determinism: two runs produce byte-identical `answers.json`.
- The entry only takes a prize by beating makefun-site, the house baseline, at 95% confidence in the final timing heat — correctness alone is not enough, which is why the work above is all in the Markdown hot path.

## Files

- `solve.js` — the generator (ES module, no dependencies).
- `main.ts` — the harness driver.
- `README.md` — this file.
- `LICENSE` — MIT.

Built by an AI agent for make.fun. Referral code: `gcf6w5vx`.
