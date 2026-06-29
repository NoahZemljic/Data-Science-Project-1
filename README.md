# ✈️ Airplane Detection with Computer Vision

Detecting and counting aircraft in top-down aerial imagery with YOLOv11, served through a lightweight, privacy-friendly web app.

![Python](https://img.shields.io/badge/Python-3.13-blue)
![Model](https://img.shields.io/badge/Model-YOLOv11-orange)
![Django](https://img.shields.io/badge/Django-5.2-092E20)
![License](https://img.shields.io/badge/License-MIT-green)

> HSLU semester project — Team 1: Noah Zemljic, Noé Felber, Dominic Zybach, Timo Ryser

---

## Overview

Airports must maintain constant awareness of aircraft positions on the ground — for runway inspection, security and regulatory compliance, gate and parking billing, and above all to prevent runway incursions, one of the most serious safety hazards in aviation. This project supports that task with a computer-vision system that **detects and counts aircraft in top-down aerial images**.

Users upload an aerial image to a web app; an object-detection model localizes every aircraft with bounding boxes, and the interface shows an interactive before/after slider alongside the total number of aircraft detected.

The project also benchmarks two models against each other: a **fine-tuned YOLOv11** model and a **YOLOv11-architecture model trained from scratch**, evaluated under identical data, splits, and metrics.

## Demo

<!-- Once you have one, drop it into results/ and uncomment: -->

![Demo](images/dashboard.gif)

<!-- Verify this EC2 instance is still running before publishing; remove this line if it has been torn down. -->

## Key results

Final fine-tuned model (**YOLOv11s**), evaluated on a held-out test set of 373 images containing 3,549 aircraft:

| Model                              | Box Precision | Recall    | mAP@50    | mAP@50–95 |
| ---------------------------------- | ------------- | --------- | --------- | --------- |
| **Fine-tuned YOLOv11s** (deployed) | **0.955**     | **0.903** | **0.934** | **0.688** |
| Custom YOLOv11 (from scratch)      | 0.888         | 0.780     | 0.822     | 0.507     |

Transfer learning wins decisively — most of all on mAP@50–95. The main weakness of the model trained from scratch is missed detections (false negatives) in dense, small-scale, or low-contrast scenes.

## Features

- Upload aerial imagery and detect every aircraft with bounding boxes
- Interactive **before/after comparison slider** to verify detections against the original
- Total **aircraft count** displayed for each image
- Download the annotated result
- Stateless by design — images are processed in memory and never stored

## Approach

### Data

Four sources were merged into a single dataset of ~3,344 images, all normalized to **640×640** and converted to a unified **YOLO label format** (single class: `airplane`):

| Source                                                     | Original format | Images |
| ---------------------------------------------------------- | --------------- | ------ |
| Roboflow — Plane Computer Vision Dataset                   | YOLOv7          | 799    |
| HRPlanesv2 (FlightScope, Zenodo)                           | YOLOv7          | 1,700  |
| Kaggle — Airplanes Dataset for R-CNN                       | CSV             | 733    |
| Manually collected (Google Maps, annotated with VisioFirm) | YOLO            | ~92    |

### Preprocessing

Images were proportionally resized with black padding to 640×640; Kaggle's absolute-pixel CSV labels were converted to normalized, center-based YOLO coordinates; Google Maps captures were hand-annotated in VisioFirm.

### Modelling

- **Fine-tuned:** several COCO-pretrained YOLOv11 variants (n, s) were trained across progressively larger dataset configurations, then hyperparameter-tuned. The final model — **YOLOv11s trained with 150 epochs, tuned hyperparameters and early stopping** — was selected based off of mAP@50–95 and deployed.
- **Custom (from scratch):** a compact, nano-scale YOLOv11-architecture model with randomly initialized weights, trained with AdamW (lr 0.002, weight decay), batch size 8, 100 epochs — on the same data and splits, as a transfer-learning baseline.

Trained with the Ultralytics framework. Evaluation metrics: Box Precision, Recall, mAP@50, mAP@50–95.

## Tech stack

**ML:** Python · Ultralytics YOLOv11 · COCO-pretrained backbones
**Web app:** Django (server + in-process inference) · Vanilla JavaScript · Bootstrap + custom CSS
**Infra:** AWS EC2 (Linux) · stateless, in-memory image processing (Base64)

## Project structure

```
Data-Science-Project-1/
├── preprocessing/        # Resizing & label conversion to a unified YOLO format
├── custom_model/         # YOLOv11-architecture model trained from scratch
├── finetuned_model/      # Fine-tuned YOLOv11 (deployed model)
├── results/              # Evaluation outputs, metrics & prediction samples
├── airplane_detection/   # Django web app
├── requirements.txt
└── README.md
```

## Getting started

### Requirements

- **Python 3.13.0** (recommended via [pyenv](https://github.com/pyenv/pyenv))
- **Django 5.2.9**

### Installation

**Linux & macOS**

```bash
pyenv install -s 3.13.0
pyenv local 3.13.0
python -V  # Python 3.13.0

python -m venv .dspro1_env
source .dspro1_env/bin/activate

python -m pip install --upgrade pip
pip install -r requirements.txt

python airplane_detection/manage.py runserver
```

**Windows (PowerShell)**

```powershell
# Ensure Python 3.13.0 is installed (https://www.python.org/downloads/)
python -V  # Python 3.13.0

python -m venv .dspro1_env
.\.dspro1_env\Scripts\Activate.ps1

python -m pip install --upgrade pip
pip install -r requirements.txt

python airplane_detection\manage.py runserver
```

Then open <http://127.0.0.1:8000>, upload a top-down aerial image, and run the detection.

## Notes

The app is stateless: uploaded images are processed in memory (Base64-encoded) and never written to disk or a database.

## License

Released under the [MIT License](LICENSE).
