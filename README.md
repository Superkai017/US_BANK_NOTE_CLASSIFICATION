# USD Banknote Denomination Classifier

A convolutional neural network that identifies the denomination of a US banknote from a single image, built with deployment on cash-handling hardware in mind.

---

## Overview

Automated cash handling systems, ATM deposit modules, teller-side counters, and accessibility applications all depend on reliably identifying what denomination a note is before any downstream logic runs. This project trains and evaluates an image classifier for that task, with an emphasis on the conditions such systems actually face: worn notes, poor lighting, partial occlusion, and off-angle capture.

The design priority is deployment realism rather than benchmark score. Public banknote datasets are captured under clean, consistent conditions, so a model that performs well on them may still fail in production. This repository therefore reports performance on a separate field test set of self-captured photographs, evaluates errors under a cost-weighted scheme that reflects the asymmetric consequences of confusing high and low denominations, and defines a confidence threshold below which predictions are deferred to manual review.

**Scope note.** This is a denomination classifier. It is not a counterfeit detection system. Counterfeit detection requires paired genuine and forged samples and typically ultraviolet or infrared imaging, none of which is present in this dataset. See [Limitations](#limitations).

---

## Dataset

| Source | Classes | Images | Notes |
|---|---|---|---|
| [USD Bill Classification Dataset](https://www.kaggle.com/datasets/aishwaryatechie/usd-bill-classification-dataset) (Kaggle) | 6 | TODO | USD 1, 5, 10, 20, 50, 100 |
| Field test set (self-captured) | 6 | TODO | Phone camera, uncontrolled conditions |

Splits are stratified by class at TODO / TODO / TODO percent for train, validation, and test. The field test set is held out entirely from training and is used only for final generalization measurement.

Raw data is not committed to this repository. Run the download script described in [Setup](#setup) to reproduce the data directory.

---

## Results

Metrics on the held-out test set:

| Metric | Value |
|---|---|
| Overall accuracy | TODO |
| Macro F1 | TODO |
| Worst-class recall | TODO |
| Inference latency (CPU, single image) | TODO ms |
| Model size | TODO MB |

Per-class performance:

| Class | Precision | Recall | Support |
|---|---|---|---|
| USD 1 | TODO | TODO | TODO |
| USD 5 | TODO | TODO | TODO |
| USD 10 | TODO | TODO | TODO |
| USD 20 | TODO | TODO | TODO |
| USD 50 | TODO | TODO | TODO |
| USD 100 | TODO | TODO | TODO |

### Cost-weighted evaluation

Accuracy alone is the wrong objective here. Misclassifying a USD 100 note as a USD 10 note is not equivalent in cost to the reverse, and neither is equivalent to confusing a 5 with a 1. The evaluation therefore reports a cost-weighted error using a denomination-difference penalty matrix, alongside a confidence threshold at which low-certainty predictions are routed to manual review rather than accepted automatically.

At a confidence threshold of TODO, the model auto-accepts TODO percent of inputs with TODO percent accuracy on that subset, deferring the remainder.

### Field performance

Clean benchmark accuracy overstates real-world reliability. Measured on self-captured photographs under uncontrolled conditions:

| Condition | Accuracy |
|---|---|
| Controlled test set | TODO |
| Field test set | TODO |

TODO: brief analysis of where the gap comes from.

---

## Architecture

TODO: state the backbone, whether weights were frozen or fine-tuned, the classification head, and the input resolution.

Augmentation during training simulates capture conditions rather than maximizing benchmark accuracy:

- Random rotation and perspective warp, for off-angle capture
- Brightness and contrast jitter, for variable lighting
- Gaussian blur and additive noise, for low-quality sensors
- Random erasing, for partial occlusion and folded notes

---

## Repository structure

```
.
├── data/                  Datasets (gitignored; see setup)
│   ├── raw/
│   ├── processed/
│   └── field/
├── notebooks/
│   ├── 01_eda.ipynb              Class balance, image statistics
│   ├── 02_training.ipynb         Model training and tuning
│   └── 03_evaluation.ipynb       Confusion matrix, cost analysis
├── src/
│   ├── data.py            Loading, splitting, augmentation
│   ├── model.py           Architecture definition
│   ├── train.py           Training loop and checkpointing
│   ├── evaluate.py        Metrics and cost-weighted analysis
│   └── predict.py         Single-image inference
├── api/
│   ├── main.py            FastAPI service
│   └── schemas.py         Request and response models
├── models/                Saved weights (gitignored)
├── reports/
│   └── figures/           Generated plots
├── requirements.txt
└── README.md
```

---

## Setup

Requires Python 3.10 or later.

```bash
git clone https://github.com/Superkai017/TODO.git
cd TODO

python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

pip install -r requirements.txt
```

Download the dataset. This requires a Kaggle API token placed at `~/.kaggle/kaggle.json`:

```bash
bash scripts/download_data.sh
```

---

## Usage

Train:

```bash
python -m src.train --config configs/baseline.yaml
```

Evaluate against the test set and regenerate all reported figures:

```bash
python -m src.evaluate --checkpoint models/best.pt --split test
```

Classify a single image:

```bash
python -m src.predict --image path/to/note.jpg
```

Serve the model:

```bash
uvicorn api.main:app --reload
```

The service exposes `POST /predict`, which accepts a multipart image upload and returns the predicted denomination, a confidence score, and a flag indicating whether the prediction falls below the manual-review threshold.

---

## Limitations

- **Not a counterfeit detector.** The model classifies denomination only. It has no capacity to distinguish genuine notes from forgeries and must not be used for authentication.
- **Series coverage.** The training data does not cover every printing series or issue year, and performance on older or redesigned notes is unmeasured.
- **Capture assumptions.** The model assumes a substantially unoccluded view of one note. Overlapping notes, stacks, and heavily damaged bills are out of scope.
- **Single currency.** USD only. The model has no defined behavior on non-USD notes and will assign them a USD class with unwarranted confidence unless an out-of-distribution check is added.
- **Benchmark optimism.** Public dataset images are cleaner than production input. The field test set exists to make that gap visible rather than to hide it.

---

## Roadmap

- [ ] Out-of-distribution rejection for non-USD and non-banknote inputs
- [ ] Multi-note detection, moving from classification to object detection
- [ ] Quantization and on-device benchmarking for embedded deployment
- [ ] Audio output for accessibility use

---

## References

TODO: dataset citation, backbone paper, and any prior work on banknote recognition you drew on.

---

## Author

Art Oudom — [github.com/Superkai017](https://github.com/Superkai017)

American University of Phnom Penh

## License

TODO: MIT is a reasonable default. Note that the underlying dataset carries its own license, which is separate from the license on this code.
