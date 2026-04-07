# VCWorld: A Biological World Model for Virtual Cell Simulation

VCWorld is a cell-level white-box simulator that integrates structured biological knowledge with LLM-based reasoning to predict cellular responses to perturbations in an interpretable, data-efficient way.

This repository provides the official implementation for VCWorld, including:
- a CLI pipeline for DE/DIR label generation, retrieval, prompt construction, and inference,
- prompt templates and single-case analysis utilities,
- inference runners for local HuggingFace (HF) models or API-backed LLMs.

![VCWorld pipeline](assets/VCWorld%20pipeline.png)

## Overview
VCWorld introduces a biological world model that explicitly reasons through mechanisms rather than relying on black-box prediction. It is designed for data-efficient, interpretable prediction of perturbation effects.

Key features:
- White-box reasoning grounded in pathways, protein interactions, and gene regulation.
- LLM-integrated inference with structured reasoning prompts.
- GeneTAK benchmark for DE and DIR prediction.
- Interpretable outputs with explicit rationale traces.

## Model Architecture
The VCWorld pipeline runs in three stages:
1) Knowledge integration: builds an open-world biological knowledge graph from public sources.
2) Evidence retrieval: finds supporting cases using semantic and graph-aware similarity.
3) Structured reasoning: synthesizes evidence to predict DE or DIR with a mechanistic explanation.

## Dataset: GeneTAK
GeneTAK is derived from the Tahoe-100M single-cell atlas and focuses on gene-level perturbation responses. You can download the constructed open world knowledge graph from [https://doi.org/10.5281/zenodo.18513982](https://doi.org/10.5281/zenodo.18513982).

- Cell lines: 5 (C32, HOP62, HepG2/C3A, Hs 766T, PANC-1)
- Perturbations: 348 drug compounds
- Tasks: Differential Expression (DE) and Directional Change (DIR)
- Format: triplets (cell line, perturbation, gene) with task-specific labels
- Splits: train/test by perturbation (30/70) to simulate few-shot conditions

## Quick Start
### Environment Setup
```bash
git clone https://github.com/GENTEL-lab/VCWorld.git
cd VCWorld
conda create -n vcworld python=3.10
conda activate vcworld
pip install -r requirements.txt
````

## CLI Pipeline (DE/DIR)

Run from `pipeline/cli_pipeline`:

```bash
cd pipeline/cli_pipeline
```

### DE example

```bash
python cli.py de prepare \
  --h5ad path/to/C32_cells.h5ad \
  --out-dir path/to/out_dir \
  --cell-line C32

python cli.py de retrieve \
  --data-csv path/to/out_dir/C32_DE.csv \
  --drug-sim path/to/combined_similarity_sorted.json \
  --gene-sim path/to/results_close_gene.json \
  --out path/to/out_dir/C32_DE_retrieval.json \
  --budget 10 --seed 42

python cli.py de prompt \
  --retrieval path/to/out_dir/C32_DE_retrieval.json \
  --template path/to/DE_template.py \
  --drug-desc path/to/drug_simp.json \
  --gene-desc path/to/gene_output.json \
  --out path/to/out_dir/C32_DE_prompts.txt

python cli.py de infer \
  --model path/to/Llama3.1-8B \
  --prompts path/to/out_dir/C32_DE_prompts.txt \
  --out path/to/out_dir/C32_DE_predictions.txt \
  --batch-size 4 --max-new-tokens 1024

python cli.py de infer-api \
  --api-url https://api.example.com/v1/chat/completions \
  --api-model your-model-name \
  --prompts path/to/out_dir/C32_DE_prompts.txt \
  --out path/to/out_dir/C32_DE_predictions_api.txt \
  --max-new-tokens 1024
```

For DIR, replace `de` with `dir` and use DIR CSV/output paths.

## Single-case analysis

Use this when the (Pert, Gene, Cell line) triple is out-of-dataset. The flow is:

1. Search drug/gene similarity JSONs.
2. If missing, optionally use an LLM to pick the most similar drug/gene from description lists.
3. Pull similar (pert, gene) pairs from the CSV as evidence examples.

Example:

```bash
python cli.py single prompt \
  --pert BMP-2 \
  --gene ALK3 \
  --cell-line C32 \
  --data-csv path/to/C32_DE.csv \
  --drug-desc path/to/drug_simp.json \
  --gene-desc path/to/gene_output.json \
  --drug-sim path/to/combined_similarity_sorted.json \
  --gene-sim path/to/results_close_gene.json \
  --out path/to/out_dir/BMP-2_ALK3_C32_single_prompt.txt \
  --mode de \
  --case-split train
```

LLM fallback (optional):
```bash
python cli.py single prompt \
  --pert BMP-2 \
  --gene ALK3 \
  --cell-line C32 \
  --data-csv path/to/C32_DE.csv \
  --drug-desc path/to/drug_simp.json \
  --gene-desc path/to/gene_output.json \
  --drug-sim path/to/combined_similarity_sorted.json \
  --gene-sim path/to/results_close_gene.json \
  --out path/to/out_dir/BMP-2_ALK3_C32_single_prompt_llm.txt \
  --mode de \
  --llm-api-url YOUR_LLM_INFERENCE_ENDPOINT \ 
  --llm-api-model MODEL_NAME \
  --llm-api-key YOUR_API_KEY
```

Notes:

* `--cell-line` must match a name in the prompt template `cell_lines` list; otherwise the first entry is used.
* `--case-split` defaults to `train`; use `all` to search across splits.
* LLM fallback runs only when the query drug/gene is missing from the similarity JSON.
* `--mode` selects DE or DIR prompt format for the single-case prompt.


## Citation

If you find VCWorld useful for your work, please cite:

```bibtex
@inproceedings{vcworld2026,
  title={VCWorld: A Biological World Model for Virtual Cell Simulation},
  author={Wei, Zhijian and Ma, Runze and Wang, Zichen and Li, Zhongmin and Song, Shuotong and Zheng, Shuangjia},
  booktitle={International Conference on Learning Representations (ICLR)},
  year={2026}
}
```
