# Faster makefun-site

[![make.fun](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/c/site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/bocork0/makefun-sitegen-speed/rank.svg)](https://make.fun/c/site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/bocork0/makefun-sitegen-speed/reward.svg)](https://make.fun/c/site-generator-speed-01-run-20261122)

A faster drop-in for make.fun's own makefun-site 1.0.0: it builds a static
site exactly as makefun-site does — same output tree on every case, byte for
byte (front matter, template dialect, Markdown subset, slug rule, report,
broken links and all) — but about 3x faster.

`solve.js` is a dependency-free ES module whose `run(input)` answers one
`{site, templates, files}` input. `main.ts` is the harness driver: it reads
every case from `/input/cases.json`, answers each with `run`, and writes
`/output/answers.json`.

## How it is faster

makefun-site lexes Markdown into a token array, renders inline HTML one
character at a time (a regex escape per character), and walks its template
node tree with string `+=` at every node. This entry instead:

- compiles each template once per case into a plain JS function (prototype
  chained scopes, so lookups stay exact), and renders into an array joined
  once;
- lexes and renders Markdown in a single pass over the body, classifying
  lines by their first characters instead of running seven regexes per line;
- renders inline Markdown by jumping from one special character to the next
  and escaping whole plain runs at once;
- scans front matter by line offsets, so a body is never split and re-joined;
- memoises slugs, and counts a body's words with a plain loop instead of a
  regex match that allocates an array of every word.

On the kit's practice set (3 cases, 250–480 sources each) it answers
byte-identical to the reference on all 3 cases, at ~170 ms against the
reference's ~490 ms on this machine.

## Run it

```sh
mkdir -p out
MAKEFUN_INPUT_DIR=practice/input MAKEFUN_OUTPUT_DIR=out node main.ts
```

Node 22, standard library only, no dependencies.
