# Sketch2Simulation

[![Lint](https://github.com/OptiMaL-PSE-Lab/Sketch2Simulation/actions/workflows/lint.yml/badge.svg)](https://github.com/OptiMaL-PSE-Lab/Sketch2Simulation/actions/workflows/lint.yml)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![arXiv](https://img.shields.io/badge/arXiv-2603.24629-b31b1b.svg)](https://arxiv.org/abs/2603.24629)

Sketch2Simulation converts a process sketch image into an executable Aspen HYSYS Python script through a multi-agent pipeline.

📄 **Paper:** [Sketch2Simulation: Automating Flowsheet Generation via Multi Agent Large Language Models](https://arxiv.org/abs/2603.24629) (arXiv:2603.24629)

> **Prerequisites:** This project requires a licensed installation of **Aspen HYSYS** with COM access enabled. Without a valid HYSYS licence the execution stage cannot run.

---

## Pipeline Overview

| Step | Agent / Module | Description |
|------|---------------|-------------|
| 1 | Descriptor | Describes the process from the input diagram image |
| 2 | Description Validator | Checks the description for repetition, inconsistency, and nonsense |
| 3 | Extractor | Converts the description into an intermediate structured representation |
| 4A | Mixing Points Normalizer | Resolves multi-input stream merges |
| 4B | Coupling Normalizer | Resolves stream coupling across unit operations |
| 5 | Basis Agent | Generates the HYSYS case setup, property package, and component section |
| 6 | Instantiation Agent | Adds unit operations and material streams |
| 7 | Unit View Normalization | Builds a unit IN/OUT view for stream connection |
| 8 | Configuration Agent | Connects streams to unit operations |
| 9 | Merge + Execute | Merges the generated script, executes it, and applies runtime-driven fixing retries |

---

## Project Structure

```text
.
├── run_workflow.ipynb              # Main orchestrator notebook
├── requirements.txt
├── images/                         # Input process sketch images
│   ├── cs1.png
│   ├── cs2.png
│   ├── cs3.png
│   └── cs4.png
└── sketch2sim/
    ├── agents/
    │   ├── descriptor_agent.py
    │   ├── description_validator_agent.py
    │   ├── extractor_agent.py
    │   ├── basis_agent.py
    │   ├── instantiation_agent.py
    │   ├── configuration_agent.py
    │   ├── fixer_agent.py
    │   └── executor_agent.py
    ├── mixing_points_normalizer.py
    ├── coupling_normalizer.py
    ├── merge_simulation_scripts.py
    ├── utils/
    │   ├── agent_utils.py
    │   ├── executor_utils.py
    │   ├── logging_utils.py
    │   └── normalization_utils.py
    ├── RAG/
    │   ├── rag_components.py
    │   ├── components_list.xlsx
    │   └── mixture_recipes.txt
    └── instructions/
        └── hysys_template_base.py
```

> **Note on instruction files:** Agent instruction files (`instantiation_instructions_*.txt`, `configuration_instructions_*.txt`) are required but not included in this repository — these contain proprietary prompt engineering for the HYSYS domain and must be sourced separately.

---

## Setup

### 1. Install dependencies

```bash
pip install -r requirements.txt
```

### 2. Configure environment variables

```bash
export OLLAMA_API_KEY=your_api_key_here
export OLLAMA_CLOUD_HOST=your_cloud_endpoint_here

# Optional — override defaults
export DESCRIPTOR_MAX_IMAGE_WIDTH=2048
export DESCRIPTOR_NUM_PREDICT=5000
export DESCRIPTOR_NUM_CTX=21000
export DESCRIPTOR_TIMEOUT=900
export EXTRACTOR_TIMEOUT=900
export EXTRACTOR_NUM_CTX=21000
```

Only `OLLAMA_API_KEY` is strictly required if the cloud endpoint is already configured in code.

### 3. Ensure Aspen HYSYS is available

A licensed installation of Aspen HYSYS must be present on the machine with COM access enabled. The executor stage communicates with HYSYS via `pywin32`. Without this, the pipeline will fail at Step 9.

### 4. Add the required instruction files

Place `instantiation_instructions_*.txt` and `configuration_instructions_*.txt` in `sketch2sim/instructions/`. These files are not distributed with this repository (see note above).

---

## Models Used

| Agent | Model |
|-------|-------|
| Descriptor | `gemini-3-flash-preview:cloud` |
| Description Validator | `qwen3-vl:8b-instruct-q4_K_M` |
| Extractor | `gemini-3-flash-preview:cloud` |
| Basis Agent | `qwen2.5-coder:latest` |
| Instantiation Agent | `qwen2.5-coder:latest` |
| Configuration Agent | `qwen3-coder:30b-a3b-q4_K_M` |
| Fixer Agent | configured via orchestrator |

---

## How to Run

1. Place your input process sketch in `images/` and update `IMAGE_PATH` in `run_workflow.ipynb` if using a different image (default: `cs1.png`).
2. Open `run_workflow.ipynb` in Jupyter and run all cells.

The pipeline writes:
- Intermediate logs to `logs/`
- Generated Python scripts to `outputs/`
- Final merged script to `outputs/hysys_combined_latest.py`

---

## Notes

- The code-generation stages rely on marker comments in `hysys_template_base.py`. If those markers are changed, the regex-based extraction and replacement logic must also be updated.
- `logs/` and `outputs/` are generated at runtime and should not be committed.
- `RAG/chroma_components/` may be generated locally and can be excluded from version control.

---

## Status

Research/prototype workflow. Requires project-specific HYSYS instruction files, a valid HYSYS licence, and appropriate model access to run end-to-end.

---

## Citation

If you use Sketch2Simulation in your research, please cite:

```bibtex
@misc{bahamdan2026sketch2simulationautomatingflowsheetgeneration,
      title={Sketch2Simulation: Automating Flowsheet Generation via Multi Agent Large Language Models},
      author={Abdullah Bahamdan and Emma Pajak and John D. Hedengren and Antonio del Rio Chanona},
      year={2026},
      eprint={2603.24629},
      archivePrefix={arXiv},
      primaryClass={cs.SE},
      url={https://arxiv.org/abs/2603.24629},
}
```
