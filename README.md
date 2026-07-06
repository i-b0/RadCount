# RadCount

Local, LLM-powered labelling of radiology reports. A browser UI lets you assign yes/no labels to free-text reports, generate those labels automatically with a local model, and score the model against your own ground truth. All inference runs on your machine via llama.cpp, so no report data leaves the computer.

![RadCount UI](docs/screenshot.png)

## Requirements

- Windows 10 or 11
- ~12 GB free disk (model and dependencies, first run only)
- An NVIDIA GPU with 8 GB+ VRAM is recommended. No GPU is fine; inference falls back to CPU, just slower.

## Setup

Automatic: double-click `setup.bat`. It installs Python if missing, then runs `startup.py` to do the rest. The comments inside `setup.bat` explain each step.

Manual: if you already have Python 3.10 to 3.12, run `python startup.py` instead.

On first run this creates a virtual environment, installs dependencies, downloads llama.cpp and the Meta-Llama-3.1-8B-Instruct model (~4.9 GB), applies migrations, and starts the server at http://127.0.0.1:8000. Later runs skip straight to launch, and interrupted downloads resume automatically. Open that URL in a browser once it starts.

## Importing reports

Reports load from a single CSV in `large_data/`. Any columns are allowed, but the last column must be the report text. See `large_data/demo_reports.csv` for the layout.

- In the UI: click Import CSV (or Replace). Export CSV downloads the current file.
- By hand: drop your `.csv` into `large_data/`. Only one CSV is kept, so a new one replaces the old.

Step through reports with the Entry number box.

## Importing and exporting annotations

All annotations and ground truth live in one `entities.json` file at the project root: a JSON array with one object per report, indexed to the CSV rows. The first entry holds the shared list of classification labels.

- In the UI: use Export Annotations / Import Annotations at the top right of the annotation panel. Export downloads the `entities.json` file; Import replaces the current annotations from an `entities.json` file you choose.
- By hand: copy an `entities.json` file into the project root (next to `manage.py`).

Keep an `entities.json` file with the CSV it was made against so the entry indices line up.

## Labelling reports

The tabs across the top are different ground-truth schemas: Binary Classification (yes/no labels), Observations, Triplets, and Entities & Relations. They share the same workflow, shown here for Binary Classification:

1. Add the labels you want under Classification Labels.
2. Mark each label yes/no for the current report in the Ground Truth column.
3. Click Extract Classifications (LLM) to auto-label the current report; results appear under Auto-Extracted.
4. Move through reports with the Entry box; each report's labels load with it.
5. Calculate Metrics runs the model across every report with ground truth and reports precision, recall, and F1.

## Changing the prompt

Every extraction task has its own Edit Prompt button next to its extract button, in each schema tab. Use it to change the template, edit label or type definitions, or switch between saved variants. Binary Classification templates must contain the `{label}` and `{report_text}` placeholders. Prompts are stored in JSON files at the project root (for example `binary_classification_prompts.json`), which you can also edit directly.

## Project structure

```
setup.bat                             Windows entry point (bootstraps Python, runs startup.py)
startup.py                            One-click setup and launch
manage.py                             Django management
radcount/                             Django project config
llm_interface/                        App: views, template, LLM service
large_data/                           Reports CSV (one at a time)
entities.json                         Labels and ground truth
binary_classification_prompts.json    Auto-labelling prompts
llm_models/                           .gguf model files (gitignored)
llama_cpp/                            llama.cpp binaries (gitignored)
```
