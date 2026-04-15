# 🔁 Web-Performance-Autoresearcher

An autonomous experiment where an LLM continuously optimizes a static website for page load performance — no human in the loop.

> **128.2ms → 11.4ms** — 91% improvement across 9 experiments

---

## How it works

Hypothesis → Modify Code → Benchmark → Evaluate → Keep or Discard → 🔁 Repeat

- ✅ Improved → `git commit`, advance the branch
- ❌ Same or worse → `git reset --hard HEAD~1`, try again

The agent runs this loop forever across 5 cold-cache Puppeteer runs per experiment. No human input needed once it starts.

---

## Repo structure

```
.
├── index.html          # ✏️  the only file the LLM edits
├── server.js           # 🔒 static file server on port 3000 (read-only)
├── benchmark.mjs       # 🔒 Puppeteer harness, 5 runs (read-only)
├── results.tsv         # 📋 experiment log (untracked by git)
├── run.log             # last benchmark output
└── public/
    └── images/         # ✏️  local assets (format/size changes allowed)
```

---

## Setup

```bash
# install dependencies
npm install puppeteer

# start the server
node server.js &

# init the experiment log
echo -e "commit\tload_time_ms\tstatus\tdescription" > results.tsv

# create a dated branch
git checkout -b autoresearch/$(date +%b%d | tr '[:upper:]' '[:lower:]')
```

---

## Running the benchmark

```bash
node benchmark.mjs
```

```
load_time_ms: 35.2
all_runs: [13.0, 19.4, 35.2, 35.3, 36.9]
```

The key metric is `load_time_ms` (median). Lower is better.

---

## Results

| commit | load_time_ms | status | description |
|--------|-------------|--------|-------------|
| 5006362 | 128.2 | ✅ keep | baseline |
| cd81f61 | 35.2 | ✅ keep | remove Google Fonts, use system fonts |
| 935a73b | 33.7 | ✅ keep | resize + JPEG images, 40–92% size reduction |
| 7c94734 | 11.4 | ✅ keep | move JS to end of body, decoding=async on images |
| c8da459 | 34.3 | ❌ discard | preload profile, add width/height to images (regression) |
| 3c113a1 | 31.5 | ❌ discard | loading=lazy on below-fold images (regression in headless) |
| 2896954 | 32.4 | ❌ discard | halve image dims to actual display size (noise) |

`results.tsv` is intentionally left untracked by git.

---

## Rules

**✅ Allowed**
- Edit `index.html` freely — HTML, inline CSS, inline JS, resource hints
- Add, remove, or convert images in `public/images/`
- Change image format, dimensions, or compression

**❌ Off-limits**
- Modify `server.js` or `benchmark.mjs`
- Fetch external resources at runtime (no CDN, no Google Fonts)
- Use a different server or port

> **Simplicity criterion:** a tiny improvement that adds complexity is not worth it. Removing something and getting equal or better results is always a win.

---

## Optimization ideas

| Technique | Notes |
|-----------|-------|
| Remove Google Fonts | Fall back to system fonts or inline `@font-face` |
| Convert PNG → JPEG/WebP | Big payload wins for photos |
| Resize images to display size | Avoid browser downscaling |
| Move JS to bottom / add `defer` | Eliminates render-blocking scripts |
| Inline critical CSS | Removes a blocking stylesheet request |
| Remove unused CSS | Reduces parse time |
| Add `decoding="async"` on images | Offloads decode from the main thread |
| Reduce DOM size | Fewer nodes = faster layout |

---

## About

Built by **Ajesh Nadar** — M.S. Data Science, ASU '26

[![LinkedIn](https://img.shields.io/badge/LinkedIn-ajesh--nadar-0A66C2?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/ajesh-nadar/)
[![Website](https://img.shields.io/badge/Website-ajesh.framer.ai-black?style=flat&logo=framer&logoColor=white)](https://ajesh.framer.ai/)
