# 🚀 Web Performance Autoresearcher

An AI-powered agent that autonomously benchmarks, analyzes, and optimizes web performance - running experiments while you sleep.

---

## 📌 Overview

The **Web Performance Autoresearcher** is an agentic system that automatically:
- Runs performance benchmarks on a web page
- Identifies bottlenecks (render-blocking resources, unoptimized images, slow scripts)
- Applies optimization experiments (image compression, JS deferral, etc.)
- Logs and compares results across experiments
- Produces a research report of what worked and what didn't

---

## 🧠 How It Works

```
index.html (target site)
     ↓
benchmark.mjs (measures performance metrics via Puppeteer/Lighthouse)
     ↓
AI Agent (analyzes results, proposes next optimization)
     ↓
server.js (serves the modified site for re-benchmarking)
     ↓
results.tsv (logs metrics per experiment)
     ↓
program.md (auto-generated research report)
```

---

## 📁 Project Structure

```
├── index.html          # Target webpage being optimized
├── server.js           # Local server to serve the webpage
├── benchmark.mjs       # Benchmarking script (measures Core Web Vitals)
├── results.tsv         # Experiment results log (tab-separated)
├── run.log             # Full agent run logs
├── program.md          # Auto-generated optimization research report
├── public/
│   └── images/         # Web assets used by index.html
└── package.json        # Node.js dependencies
```

---

## ⚙️ Setup & Installation

### Prerequisites
- Node.js v18+
- npm

### Install dependencies
```bash
npm install
```

### Run the server
```bash
node server.js
```

### Run benchmarks
```bash
node benchmark.mjs
```

---

## 📊 Experiments Conducted

| Experiment | Optimization Applied | Result |
|---|---|---|
| Baseline | No changes | Slow load time |
| exp2 | Resize + JPEG compression (40–92% size reduction) | ✅ Improved |
| exp3 | Move JS to end of body + `decoding=async` on images | ✅ Improved |

> Full results available in [`results.tsv`](./results.tsv)

---

## 🛠️ Tech Stack

- **Node.js** — Runtime
- **JavaScript / ES Modules** — Benchmarking logic
- **HTML/CSS** — Target webpage
- **Puppeteer / Lighthouse** — Performance measurement
- **AI Agent** — Experiment planning and analysis

---

## 📄 Research Report

The agent auto-generates a research report after each run. See [`program.md`](./program.md) for the full findings.

---

## 👤 Author

**Ajesh Thangaraj Nadar**
- Portfolio: [Website](https://ajesh.framer.ai/)
- LinkedIn: [linkedin.com/in/ajesh-nadar](https://linkedin.com/in/ajesh-nadar)

---

## 📃 License

MIT License — feel free to use and build on this project.
