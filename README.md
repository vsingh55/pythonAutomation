# ETAP Automated PDF Report Generator

This repository contains a modular Python automation pipeline designed to parse operational parameter logs (from ETAP simulation run routines, exported as `.xlsx` or `.csv` files) and dynamically generate publication-quality, classic academic conference-style PDF reports.

## Features

- **Lightweight & Fast**: Built using pure-Python libraries (**ReportLab** & **Matplotlib**). Runs in milliseconds without heavy system dependencies (no headless browsers like Playwright/Chromium or HTML-to-PDF compilers like WeasyPrint/wkhtmltopdf).
- **Classic Academic Styling**: Adheres to a monochrome greyscale IEEE-style format including title sections, authors, dates, structured tables, mathematical limits status (PASS/FAIL/WARNING highlight), and running headers/footers with dynamic page numbers (`Page X of Y`).
- **Data Visualizations**: Matplotlib plots are auto-generated from incoming time-series datasets:
  1. **Line Chart**: Trend runs of parameters across the operational period.
  2. **Bar Chart**: Direct comparative analysis of final measured parameter values against safety limits.
- **Dual Execution Modes**:
  1. **CLI Mode**: Run a command to compile a single report from a path.
  2. **Watcher Mode**: Run a lightweight background monitoring service that automatically processes any new spreadsheets dropped into the input folder.

---

## Directory Structure

```text
├── data/
│   ├── input/             # Place incoming spreadsheets here (.xlsx / .csv)
│   └── output/            # Contains completed PDF reports and temporary charts
├── logs/
│   └── automation.log     # Detailed pipeline logs and execution traces
├── src/
│   ├── parser.py          # Handles Excel & CSV reading, metadata, and tables
│   ├── charts.py          # Builds Matplotlib monochrome visualizations
│   ├── generator.py       # Renders the ReportLab PDF pages and custom canvas layouts
│   └── main.py            # Orchestrator for CLI arguments, logging, and Watchdog daemon
├── requirements.txt       # Python package requirements
└── README.md              # Project documentation
```

---

## Setup & Installation

1. Create a Python virtual environment:
   ```bash
   python3 -m venv .venv
   ```
2. Activate and install requirements:
   ```bash
   .venv/bin/pip install -r requirements.txt
   ```

---

## Usage

Ensure you run commands with `PYTHONPATH=src` set (or add `src/` to your environment path) to resolve local module imports correctly.

### 1. Compile a Specific Report (CLI Mode)
To compile a spreadsheet into a PDF report immediately:
```bash
PYTHONPATH=src .venv/bin/python src/main.py --input data/input/sample_run_01.xlsx
```
You can also explicitly define the destination PDF path:
```bash
PYTHONPATH=src .venv/bin/python src/main.py --input data/input/sample_run_02.csv --output data/output/turbine_compliance_report.pdf
```

### 2. Auto-Process New Files (Watcher Mode)
To launch the background folder watcher monitoring the `data/input/` directory:
```bash
PYTHONPATH=src .venv/bin/python src/main.py --watch
```
Drop any new `.csv` or `.xlsx` file into `data/input/`, and the PDF report will automatically be created in `data/output/` within seconds. Press `Ctrl+C` in the terminal to stop the watcher daemon.

### 3. Live Demo Portal (Web Portal Mode)
To launch a self-hosted Flask portal where anyone can upload spreadsheets and download generated PDFs directly in their browser:
```bash
PYTHONPATH=src .venv/bin/python src/app.py
```
By default, the server runs on port `5000` (e.g., `http://localhost:5000/` or `http://<your-homelab-ip>:5000/`).

---

## Input Spreadsheet Layout Formats

### Format A: Multi-sheet Excel (.xlsx)
Create an Excel file with exactly three sheets:
1. `Metadata`: Key-value rows (A: parameter key, B: text value) for `Title`, `Author`, `Date`, `Abstract`, and `Methodology`.
2. `Summary`: Tabular columns: `Parameter`, `Measured`, `Units`, `Limit`, `Status`, `Conclusion`.
3. `TimeSeries`: Row-column table with a time index in the first column, and measurements for the parameters in remaining columns.

### Format B: Marked Block CSV (.csv or single-sheet Excel)
Provide a flat file separated into blocks using section markers `[Metadata]`, `[Summary]`, and `[TimeSeries]`:
```csv
[Metadata]
Title,Primary Utility Inverter Run
Author,Vijay Kumar Singh
Date,2026-06-18
Abstract,Brief summary of the test.
Methodology,Step load testing.

[Summary]
Parameter,Measured,Units,Limit,Status,Conclusion
Active Power,45.2,kW,>40,PASS,Normal feed-in.

[TimeSeries]
Time,Active Power
0,40.1
10,42.5
20,44.8
```

---

## Running Regression Tests

The project includes a regression test suite that verifies parsing logic (Excel & CSV), chart generation, PDF compiling, and the end-to-end processing pipeline:

```bash
.venv/bin/python -m unittest discover -s tests
```

