# ECE6252 — Adversarial Robustness Evaluation on ImageNet

Empirical evaluation of adversarial robustness across five ImageNet models of increasing defense sophistication, conducted as part of the ECE6252 course project at Georgia Institute of Technology.

## Models Evaluated

| Model | Defense | Source |
|-------|---------|--------|
| Standard ResNet-50 | None (baseline) | torchvision |
| Wong2020Fast | FGSM adversarial training | RobustBench |
| Engstrom2019Robustness | PGD adversarial training | RobustBench |
| Singh2023Revisiting_ConvNeXt-S-ConvStem | Augmentation + architecture + APGD adversarial training | RobustBench |
| Amini2024MeanSparse_Swin-L | Inference-time feature sparsification | RobustBench |

## Evaluation Conditions

| Condition | Details |
|-----------|---------|
| Clean | ImageNet val, 50k images |
| FGSM | ε=4/255 |
| PGD-20 | ε=4/255, α=1/255, 20 steps |
| AutoAttack | ε=4/255, 5000 balanced images (5 per class, all 1000 classes) |
| ImageNet-A | Naturally hard images, 200-class subset |

All conditions use ε=4/255 under the ℓ∞ norm.

## Requirements

- NVIDIA GPU with at least 40GB VRAM (evaluated on H200 HGX via PACE-ICE)
- Python 3.10+

```bash
pip install torch torchvision robustbench torchattacks autoattack \
            datasets huggingface_hub tqdm matplotlib numpy pandas
```

## Environment Setup

Set the following environment variables before launching Jupyter:

```bash
export HF_TOKEN=hf_...               # Hugging Face auth token
export SCRATCH=/your/scratch/path    # Cache directory 
export SCRATCH_TMP=/your/tmp/path    # Temp directory
export SAVE_DIR=/your/autoattack/tensors/path  # AutoAttack tensor storage
```

**Important:** Cell 1 must be run first every session before any other cell. It redirects all Hugging Face cache and temp files to scratch storage to avoid home directory quota issues.

## Reproducing Results

Run cells top-to-bottom in order:

1. **Cell 1** — Environment setup (run first every session)
2. **Cell 2** — Imports and Hugging Face login
3. **Cell 3** — Define transforms
4. **Cell 4** — Load all five models
5. **Cell 5** — Define evaluation functions
6. **Cells 6–10** — Clean accuracy evaluations
7. **Cells 11–15** — FGSM attack evaluations
8. **Cells 16–20** — PGD attack evaluations (compute-intensive)
9. **Cells 21–27** — AutoAttack evaluations (compute-intensive)
10. **Cells 28–32** — ImageNet-A evaluations
11. **Final cells** — Results table and plots

Results are saved to `all_results.json` after every evaluation. It is safe to restart the kernel mid-run — completed evaluations will not be re-run as long as `all_results.json` is present.

> **Note:** Full re-execution requires significant GPU compute. AutoAttack and PGD evaluations on 50k images can each take up to 12 wall-clock hours on an H200 GPU.

## Datasets

- **ImageNet validation set**: [ILSVRC/imagenet-1k on Hugging Face](https://huggingface.co/datasets/ILSVRC/imagenet-1k) (requires HF account)
- **ImageNet-A**: [hendrycks/natural-adv-examples](https://people.eecs.berkeley.edu/~hendrycks/imagenet-a.tar) — downloaded automatically by the notebook

## Results

| Condition | Baseline | Wong2020 | Engstrom2019 | Singh2023 | MeanSparse |
|-----------|----------|----------|--------------|-----------|------------|
| Clean | 76.14% | 53.81% | 62.40% | 73.36% | 78.05% |
| FGSM | 27.40% | 32.69% | 39.73% | 55.55% | 66.18% |
| PGD-20 | 1.14% | 27.88% | 33.22% | 52.77% | 62.85% |
| AutoAttack | 0.70% | 24.66% | 28.32% | 50.38% | 60.26% |
| ImageNet-A | 0.00% | 0.53% | 0.65% | 2.81% | 8.21% |
