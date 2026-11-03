# MARGIN: PET-free Mapping of Active Pelvic Bone Marrow from CT and Blood Tests

**MARGIN** (Marrow Active Region Generative Identification Network) estimates PET-defined active bone marrow (ABM) in the pelvis from a radiotherapy planning CT, routine blood counts (WBC, PLT, Hb), age and sex. PET is required only for training; at inference the model needs CT and clinical variables alone.

MARGIN has three components:

1. **CT-to-pseudo-PET synthesis** (CycleGAN) that produces a PET-like image from planning CT.
2. **Hematology-guided conditional HVAE** that conditions the pseudo-PET representation on blood counts and demographics within a structural causal model, allowing single-variable blood-test interventions.
3. **Active/inactive marrow disentanglement** (contrastive, Wasserstein-critic and pseudo-IBM adversarial losses) that separates ABM from inactive marrow features.

The model was developed on one cohort and evaluated on four external cohorts covering three hospital sites and two public datasets:

| Cohort | Source | Studies | Cancer types |
|---|---|---|---|
| DEV#1 | Sun Yat-sen University Cancer Center, YueXiu campus | 1,076 | NPC, CRC |
| EX#1 | Sun Yat-sen University Cancer Center, HuangPu campus | 540 | NPC |
| EX#2 | Jiangmen Central Hospital | 203 | NPC |
| EX#3 | Soft-tissue-Sarcoma (public) | 25 | STS |
| EX#4 | FDG-PET-CT-Lesions (public) | 734 | NSCLC, melanoma |

MARGIN achieved a mean regional Dice of 83.2% on DEV#1 and 73.5–81.0% on the external cohorts.

> **Disclaimer.** MARGIN is a research tool. Its outputs are model predictions, not measurements of marrow function, and have not been validated prospectively. It is not intended for clinical decision-making. Any use in patient care is the sole responsibility of the user, and the authors provide the code without warranty of any kind, express or implied.

## Study overview

<img width="539" height="684" alt="image" src="https://github.com/user-attachments/assets/7f96f75f-36be-4d4d-8dd7-337d9dae9f29" />



<!-- Replace with Fig. 1 of the paper, saved as figures/overview.png -->

## Repository structure

```
MARGIN/
├── configs/            # training and inference configuration files
├── data/
│   └── example/        # de-identified example case (CT + clinical variables)
├── margin/             # model code (CycleGAN, conditional HVAE, disentanglement)
├── scripts/
│   ├── train.py        # training
│   ├── infer.py        # ABM prediction from CT + blood tests
│   └── intervene.py    # single-variable blood-test interventions
├── checkpoints/        # pretrained weights (see "Pretrained model")
├── requirements.txt
└── README.md
```

<!-- Edit this tree to match the files you actually upload. -->

## Requirements

- Python [3.x]
- PyTorch [x.x] with CUDA [x.x]
- See `requirements.txt` for the full list of packages and versions.

Experiments in the paper were run on four NVIDIA RTX A6000 GPUs. Inference runs on a single GPU.

```bash
git clone https://github.com/xinzhen-lab/[REPO-NAME].git
cd [REPO-NAME]
pip install -r requirements.txt
```

## Data preparation

Each case requires:

- **Planning CT** (DICOM or NIfTI).
- **Clinical variables**: WBC (×10⁹/L), PLT (×10⁹/L), Hb (g/L), age (years) and sex. Enter them in `data/example/clinical.csv`, one row per case.
- **For training only**: paired ¹⁸F-FDG PET, registered to CT, and the PET-defined ABM mask.

Preprocessing follows the paper: PET is converted to SUV and clipped to 0.01–7, PET is registered to CT with CT as the fixed image, and both are resampled to 192 × 192 pixels in the transverse plane.

## Pretrained model

Pretrained weights trained on DEV#1 are available at [link]. Place them in `checkpoints/`.

## Usage

**Predict ABM from CT and blood tests**

```bash
python scripts/infer.py --ct data/example/ct.nii.gz --clinical data/example/clinical.csv --checkpoint checkpoints/margin.pth --output results/
```

The script writes the pseudo-PET image and the predicted ABM mask for each case.

**Simulate a blood-test change**

```bash
python scripts/intervene.py --ct data/example/ct.nii.gz --clinical data/example/clinical.csv --variable WBC --value 6.5 --checkpoint checkpoints/margin.pth --output results/
```

The output is the model-predicted ABM map after setting one blood-test variable to a new value with the others held fixed. These maps are model predictions, not observed treatment effects.

**Train**

```bash
python scripts/train.py --config configs/train.yaml
```

<!-- Replace the commands and arguments above with those of your actual scripts. -->

## Data availability

Patient data from DEV#1, EX#1 and EX#2 are not publicly available because of ethics restrictions. De-identified data may be made available to qualified researchers on reasonable request to the corresponding authors, subject to ethics approval and a data use agreement. EX#3 and EX#4 are public datasets: [Soft-tissue-Sarcoma dataset link] and [FDG-PET-CT-Lesions dataset link].

## Citation

If you use this code, please cite:

```
[Authors]. Generative mapping of active pelvic bone marrow from computed tomography and blood measurements. [Journal], [year]. [DOI]
```

## License

This code is released for academic and research use only. See `LICENSE` for details. For commercial use or other inquiries, please contact Prof. Xin Zhen (xinzhen@smu.edu.cn).

## Note

- The code is provided as research code and may contain bugs. It will be updated periodically.
- Questions and bug reports are welcome via GitHub Issues.
