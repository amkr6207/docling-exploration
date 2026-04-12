# Outreachy 2026 - Issue #122

## Docling: Explore Document Processing Basics

This repository documents my hands-on exploration for:
- [Outreachy 2026] Docling: explore document processing basics (#122)

Goal: use Docling CLI to convert a PDF to multiple formats, try additional options, and compare outcomes for RAG-style preprocessing.

## Introduction

Docling is a document-conversion toolkit commonly used in Retrieval-Augmented Generation (RAG) workflows to transform source files (such as PDFs) into structured formats that are easier to parse, chunk, and index.

In this exploration, I used the Docling CLI on a brochure-style PDF and tested multiple conversion settings. The objective was to understand how output format and processing options (OCR and table extraction mode) affect:
- output structure and readability
- extraction quality
- processing behavior and runtime

This README captures the exact commands, logs, screenshots, generated artifacts, and a comparison of findings from each experiment.

## Environment Setup

### 1) Create and activate virtual environment

```bash
python3 -m venv docling-env
source docling-env/bin/activate
```

### 2) Install Docling

```bash
pip install docling
```

Installation completed successfully with dependencies including `torch`, `torchvision`, `rapidocr`, and others.

![Docling installation output](screenshots/01-install-docling.png)

### 3) Verify installed version

```bash
docling --version
```

Observed output:

```text
Docling version: 2.85.0
Docling Core version: 2.72.0
Docling IBM Models version: 3.13.0
Docling Parse version: 5.7.0
Python: cpython-312 (3.12.3)
Platform: Linux-6.17.0-20-generic-x86_64-with-glibc2.39
```

![Docling version output](screenshots/02-docling-version.png)

### 4) Check CLI options

```bash
docling --help
```

Relevant options used in this exploration:
- `--to [md|html|json|...]`
- `--device [auto|cpu|cuda|...]`
- `--no-ocr`
- `--table-mode [fast|accurate]`
- `--output PATH`

## Input Document

I used a brochure-style PDF with mixed layout content:

- `pdf/pytorch-conference.pdf`

Reason for selection:
- Multi-section layout
- Visual elements
- Better for testing parser behavior than plain text-only documents

## Experiments

All commands were run from project root:
- `/home/aman/OpenSource/outreachy/docling-exploration`

Logs were captured with `tee` into `logs/`.

---

### Experiment 1 - Default conversion (Markdown)

Command:

```bash
docling ./pdf/pytorch-conference.pdf --device cpu --output ./outputs/default 2>&1 | tee ./logs/03-default.txt
```

Primary observations from log:
- RapidOCR used CPU device.
- First run downloaded OCR model artifacts.
- Warning shown about unauthenticated HF hub requests.
- Final warning: `RapidOCR returned empty result!`

Generated output:
- `outputs/default/pytorch-conference.md`

Evidence:
- `logs/03-default.txt`

Timing note (from log timestamps):
- Approx. 3m53s (`23:40:54` to `23:44:47`), including first-time model download.

Screenshots:
- Terminal output: `screenshots/03-default-terminal.png`
- Markdown output (`pytorch-conference.md`): `screenshots/04-default-markdown-output.png`

![Experiment 1 terminal output](screenshots/03-default-terminal.png)
![Experiment 1 markdown output](screenshots/04-default-markdown-output.png)

---

### Experiment 2 - HTML conversion

Command:

```bash
docling ./pdf/pytorch-conference.pdf --device cpu --to html --output ./outputs/html 2>&1 | tee ./logs/04-html.txt
```

Primary observations from log:
- OCR model files were reused (`File exists and is valid`).
- CPU device used.
- Final warning: `RapidOCR returned empty result!`

Generated output:
- `outputs/html/pytorch-conference.html`

Evidence:
- `logs/04-html.txt`

Timing note (from log timestamps):
- Approx. 58s (`23:58:29` to `23:59:27`).

Screenshots:
- Terminal output: `screenshots/05-html-terminal.png`
- HTML output preview: `screenshots/06-html-output.png`

![Experiment 2 terminal output](screenshots/05-html-terminal.png)
![Experiment 2 HTML output](screenshots/06-html-output.png)

---

### Experiment 3 - Disable OCR

Command:

```bash
docling ./pdf/pytorch-conference.pdf --device cpu --no-ocr --output ./outputs/no-ocr 2>&1 | tee ./logs/05-no-ocr.txt
```

Primary observations from log:
- Run completed successfully.
- Log mainly shows model weight loading progress.
- No `RapidOCR returned empty result!` warning in this run.

Generated output:
- `outputs/no-ocr/pytorch-conference.md`

Evidence:
- `logs/05-no-ocr.txt`

Screenshots:
- Terminal output: `screenshots/07-no-ocr-terminal.png`
- Markdown output: `screenshots/08-no-ocr-output.png`

![Experiment 3 terminal output](screenshots/07-no-ocr-terminal.png)
![Experiment 3 markdown output](screenshots/08-no-ocr-output.png)

---

### Experiment 4 - Table mode fast

Command:

```bash
docling ./pdf/pytorch-conference.pdf --device cpu --table-mode fast --output ./outputs/table-fast 2>&1 | tee ./logs/06-table-fast.txt
```

Primary observations from log:
- CPU device used.
- OCR files reused from cache.
- Final warning: `RapidOCR returned empty result!`

Generated output:
- `outputs/table-fast/pytorch-conference.md`

Evidence:
- `logs/06-table-fast.txt`

Timing note (from log timestamps):
- Approx. 1m05s (`00:05:02` to `00:06:07`).

Screenshots:
- Terminal output: `screenshots/09-table-fast-terminal.png`
- Markdown output: `screenshots/10-table-fast-output.png`

![Experiment 4 terminal output](screenshots/09-table-fast-terminal.png)
![Experiment 4 markdown output](screenshots/10-table-fast-output.png)

---

### Experiment 5 - Table mode accurate

Command:

```bash
docling ./pdf/pytorch-conference.pdf --device cpu --table-mode accurate --output ./outputs/table-accurate 2>&1 | tee ./logs/06-table-accurate.txt
```

Primary observations from log:
- CPU device used.
- OCR files reused from cache.
- Final warning: `RapidOCR returned empty result!`

Generated output:
- `outputs/table-accurate/pytorch-conference.md`

Evidence:
- `logs/06-table-accurate.txt`

Timing note (from log timestamps):
- Approx. 54s (`00:17:57` to `00:18:51`).

Screenshots:
- Terminal output: `screenshots/11-table-accurate-terminal.png`
- Markdown output: `screenshots/12-table-accurate-output.png`

![Experiment 5 terminal output](screenshots/11-table-accurate-terminal.png)
![Experiment 5 markdown output](screenshots/12-table-accurate-output.png)

## Comparative Analysis

| Experiment | Option Tested | Output File | Approx. Time | Key Observation | Trade-off / Recommendation |
|---|---|---|---|---|---|
| 1 | Default (`--to md`) | `outputs/default/pytorch-conference.md` | 3m53s (first run) | Includes first-time OCR model download; warning `RapidOCR returned empty result!` | Good baseline run; first run may be slower due to model download/cache warm-up |
| 2 | HTML (`--to html`) | `outputs/html/pytorch-conference.html` | 58s | OCR cache reused; same OCR warning observed | Better for visual layout review; markdown remains easier for text/RAG pipelines |
| 3 | OCR disabled (`--no-ocr`) | `outputs/no-ocr/pytorch-conference.md` | N/A (runtime not captured in log) | No `RapidOCR returned empty result!` warning in this run | For this PDF, output remained usable without OCR; keep OCR for scanned/image-heavy PDFs |
| 4 | Table mode fast (`--table-mode fast`) | `outputs/table-fast/pytorch-conference.md` | 1m05s | OCR cache reused; similar structure quality in sampled sections | Use `fast` for quicker iteration/batch testing |
| 5 | Table mode accurate (`--table-mode accurate`) | `outputs/table-accurate/pytorch-conference.md` | 54s | Similar quality in sampled sections; not slower in this run | Use `accurate` when table fidelity matters most; runtime may vary by file/cache |

### Default Markdown vs HTML
| Aspect | Default Markdown (`--to md`) | HTML (`--to html`) | Conclusion |
|---|---|---|---|
| Readability for downstream text use | Simpler/plain structure, easier to parse for RAG-style pipelines | More presentation-oriented structure | Prefer markdown for text-centric preprocessing |
| Visual/layout review | Limited visual fidelity | Better for visual inspection of structure/content flow | Prefer HTML for manual review |
| Runtime impact (this dataset) | Baseline | Similar after cache warm-up | Output format itself did not create major runtime differences here |

### OCR enabled (default) vs `--no-ocr`
| Aspect | OCR Enabled (default) | OCR Disabled (`--no-ocr`) | Conclusion |
|---|---|---|---|
| Output usability for this PDF | Usable output, but OCR warning appeared | Usable output, warning absent in this run | For this source PDF, OCR did not materially improve extraction |
| Warning behavior | `RapidOCR returned empty result!` seen repeatedly | Warning not observed in this run | Suggests limited extra OCR-detectable text in this file |
| Best-fit scenario | Important for scanned/image-heavy docs | Useful for born-digital PDFs with selectable text | Keep OCR for scanned docs; `--no-ocr` can be acceptable for text PDFs |

### `--table-mode fast` vs `--table-mode accurate`
| Aspect | `--table-mode fast` | `--table-mode accurate` | Conclusion |
|---|---|---|---|
| Runtime in this run | 1m05s | 54s | `accurate` was not slower in this specific run |
| Reason runtime may differ | Subject to run-to-run variance and cache state | Subject to run-to-run variance and cache state | Compare multiple runs before generalizing performance |
| Observed output quality (sampled sections) | Broadly similar markdown structure | Broadly similar markdown structure | For this PDF, quality looked similar |
| Recommendation | Better for quick iteration/batch runs | Better when table fidelity is critical | Pick mode based on project priority: speed vs fidelity |

## Notes on CPU/GPU

I explicitly used:

```bash
--device cpu
```

This keeps runs deterministic on my laptop and avoids CUDA/driver mismatch issues seen in environment checks.

## Repository Structure

```text
docling-exploration/
├── README.md
├── pdf/
│   └── pytorch-conference.pdf
├── logs/
│   ├── 03-default.txt
│   ├── 04-html.txt
│   ├── 05-no-ocr.txt
│   ├── 06-table-fast.txt
│   └── 06-table-accurate.txt
├── outputs/
│   ├── default/
│   │   └── pytorch-conference.md
│   ├── html/
│   │   └── pytorch-conference.html
│   ├── no-ocr/
│   │   └── pytorch-conference.md
│   ├── table-fast/
│   │   └── pytorch-conference.md
│   └── table-accurate/
│       └── pytorch-conference.md
└── screenshots/
    ├── 01-install-docling.png
    ├── 02-docling-version.png
    ├── 03-default-terminal.png
    ├── 04-default-markdown-output.png
    ├── 05-html-terminal.png
    ├── 06-html-output.png
    ├── 07-no-ocr-terminal.png
    ├── 08-no-ocr-output.png
    ├── 09-table-fast-terminal.png
    ├── 10-table-fast-output.png
    ├── 11-table-accurate-terminal.png
    └── 12-table-accurate-output.png
```

## Command Summary

```bash
# Version and help
docling --version
docling --help

# Default markdown
docling ./pdf/pytorch-conference.pdf --device cpu --output ./outputs/default 2>&1 | tee ./logs/03-default.txt

# HTML
docling ./pdf/pytorch-conference.pdf --device cpu --to html --output ./outputs/html 2>&1 | tee ./logs/04-html.txt

# No OCR
docling ./pdf/pytorch-conference.pdf --device cpu --no-ocr --output ./outputs/no-ocr 2>&1 | tee ./logs/05-no-ocr.txt

# Table mode fast
docling ./pdf/pytorch-conference.pdf --device cpu --table-mode fast --output ./outputs/table-fast 2>&1 | tee ./logs/06-table-fast.txt

# Table mode accurate
docling ./pdf/pytorch-conference.pdf --device cpu --table-mode accurate --output ./outputs/table-accurate 2>&1 | tee ./logs/06-table-accurate.txt
```
