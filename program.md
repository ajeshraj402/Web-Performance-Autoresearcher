# autoresearcher/website

This is an experiment to have the LLM autonomously optimize a website for load performance.

## Setup

To set up a new experiment, work with the user to:

1. **Agree on a run tag**: propose a tag based on today's date (e.g. `mar30`). The branch `autoresearch/<tag>` must not already exist — this is a fresh run.
2. **Create the branch**: `git checkout -b autoresearch/<tag>` from current master.
3. **Read the in-scope files**: The repo is small. Read these files for full context:
   - `program.md` — this file.
   - `index.html` — the only file you modify. All HTML, CSS, and JS live here.
   - `server.js` — fixed static file server. Do not modify.
   - `benchmark.mjs` — fixed evaluation harness. Do not modify.
4. **Verify the server is running**: The benchmark assumes a server on port 3000. If not already running, start it: `node server.js &`
5. **Initialize results.tsv**: Create `results.tsv` with just the header row. The baseline will be recorded after the first run.
6. **Confirm and go**: Confirm setup looks good.

Once you get confirmation, kick off the experimentation.

## Experimentation

Each experiment modifies the website and measures page load time using Puppeteer with cache disabled across 5 runs.

**What you CAN do:**
- Modify `index.html` — this is the only file you edit. Everything is fair game: HTML structure, inline CSS, inline JS, image references, resource hints, font loading, layout, etc.
- Add, remove, or swap images in `public/images/`. Images must remain locally served (no CDN or external URLs).
- Change image formats, dimensions, or compression — but keep them as actual files on disk (the benchmark doesn't care about format, only load time).

**What you CANNOT do:**
- Modify `server.js`. It is read-only.
- Modify `benchmark.mjs`. It is the fixed evaluation harness.
- Use a different server or port. The benchmark always hits `http://localhost:3000`.
- Fetch resources from external URLs at runtime (fonts from Google Fonts, CDN scripts, etc.) — everything must be self-hosted or inlined.

**The goal is simple: get the lowest `load_time_ms` (median across 5 runs).** Since the benchmark always runs 5 cold-cache loads, you don't need to worry about cache warming. Everything is fair game: inline critical CSS, compress images, remove blocking scripts, lazy-load below-fold content, reduce DOM size, eliminate render-blocking fonts, etc.

**Simplicity criterion**: All else being equal, simpler is better. A tiny improvement that adds ugly complexity is not worth it. Removing something and getting equal or better results is a win. When evaluating whether to keep a change, weigh the complexity cost against the improvement magnitude.

**The first run**: Your very first run should always establish the baseline — run the benchmark against the unmodified site and record it.

## Running the benchmark

```
node benchmark.mjs
```

Output format:
```
load_time_ms: 82.5
all_runs: [79.7, 80.5, 82.5, 83.6, 141.2]
```

The key metric is `load_time_ms` (median). Lower is better.

## Logging results

When an experiment is done, log it to `results.tsv` (tab-separated, NOT comma-separated).

The TSV has a header row and 4 columns:

```
commit	load_time_ms	status	description
```

1. git commit hash (short, 7 chars)
2. median `load_time_ms` achieved (e.g. `82.5`) — use `0.0` for crashes
3. status: `keep`, `discard`, or `crash`
4. short text description of what this experiment tried

Example:

```
commit	load_time_ms	status	description
a1b2c3d	82.5	keep	baseline
b2c3d4e	61.2	keep	compress images to JPEG, remove Google Fonts
c3d4e5f	84.1	discard	added preload hints (regressed)
d4e5f6g	0.0	crash	broken HTML (server 500)
```

Do not commit `results.tsv` — leave it untracked by git.

## The experiment loop

The experiment runs on a dedicated branch (e.g. `autoresearch/mar30`).

LOOP FOREVER:

1. Look at the git state: the current branch/commit we're on.
2. Modify `index.html` (and optionally images) with an experimental idea.
3. `git commit` the change.
4. Run the benchmark: `node benchmark.mjs > run.log 2>&1`
5. Read the result: `grep "load_time_ms" run.log`
6. If the grep output is empty, the run crashed. Run `cat run.log` to read the error and attempt a fix. If you can't recover after a few attempts, discard and revert.
7. Record the result in `results.tsv`.
8. If `load_time_ms` improved (lower), keep the commit and advance the branch.
9. If `load_time_ms` is equal or worse, `git reset --hard HEAD~1` to revert.

**NEVER STOP**: Once the experiment loop has begun, do NOT pause to ask the human if you should continue. The human might be asleep. You are autonomous. Run until manually interrupted.

**Crashes**: If the benchmark crashes (broken HTML, server error, Puppeteer timeout), check `run.log`. If it's a trivial fix (typo, unclosed tag), fix and re-run. If the idea is fundamentally broken, log "crash" and move on.

**Timeout**: Each benchmark run takes ~10–30 seconds. If a run exceeds 2 minutes, kill it and treat it as a crash.

**Ideas to try (not exhaustive):**
- Convert PNG images to JPEG or WebP and serve locally
- Remove or self-host Google Fonts (inline font-face with base64, or fall back to system fonts)
- Move non-critical JS to bottom of `<body>` or add `defer`
- Remove unused CSS
- Reduce image dimensions to actual display size
- Inline critical CSS, defer the rest
- Reduce total DOM node count
- Remove the 3D tilt JS effect
- Use `font-display: swap` on web fonts
- Reduce number of images loaded above the fold
- Eliminate all render-blocking resources
