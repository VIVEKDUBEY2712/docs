# LeafCare AI — Plant Leaf Disease Detection using Machine Learning

A B.Tech CSE MVP that classifies supported plant leaf disease classes from an image. The project uses TensorFlow/Keras, MobileNetV2 transfer learning, a custom CNN baseline, Grad-CAM, and a Flask web interface.

> **Important:** The repository does not include the PlantVillage dataset or a trained model. The website can be previewed before training, but genuine predictions require training a model first. The app intentionally does not generate fake predictions.

## MVP features

- Responsive website with home, upload, result, and project information pages.
- JPG/JPEG/PNG upload, drag and drop, preview, file size checks, and mobile camera capture where the browser supports it.
- Flask web backend and `/api/predict` REST endpoint.
- MobileNetV2 transfer-learning model and a custom CNN baseline.
- Top-3 predictions and model confidence.
- Configurable low-confidence rejection (`CONFIDENCE_THRESHOLD`, default `0.55`).
- Grad-CAM heatmap attempt when a compatible trained model is available.
- Disease cause, symptoms, and general treatment information for the configured PlantVillage classes.
- Validation accuracy, weighted precision/recall/F1, and confusion-matrix export after training.
- Docker configuration and pytest tests.

## Synopsis alignment

| Synopsis requirement | MVP status |
|---|---|
| PlantVillage dataset | Dataset downloader is provided; dataset must be downloaded separately. |
| Field-captured images | Supported as an extension to the dataset; not included in this ZIP. |
| Image resizing, normalization, augmentation | Implemented in the training and inference pipeline. |
| MobileNetV2 transfer learning | Implemented. |
| Custom CNN baseline | Implemented; train separately for comparison. |
| ResNet50 alternative | Not included in this MVP; MobileNetV2 is the selected transfer-learning model. |
| Accuracy, precision, recall, F1-score | Validation metrics are saved after training. |
| Confusion matrix | Saved as a CSV after training. |
| Flask website and REST API | Implemented. |
| Upload and mobile camera | Upload and mobile browser capture are supported. |
| Confidence score and Top-3 results | Implemented when a trained model is present. |
| Grad-CAM | Implemented as a best-effort visualization; verify it with the trained model. |
| Treatment and prevention tips | Disease information currently includes cause, symptoms, and treatment. Some entries may need review and a prevention field can be expanded. |
| Reject blurry / non-leaf images | Invalid images and low-confidence predictions are rejected. A reliable dedicated non-leaf classifier and robust blur detector are not included yet. |
| Accuracy above 90% | This is a target, not a guarantee. Report only metrics produced by your actual training run. |

## Preview the UI without setup

Open `preview.html` directly in a browser to see the static homepage design. This is a visual preview only; image prediction needs the Flask app and a trained model.

## 1. Requirements

- Python 3.10 recommended for the pinned TensorFlow 2.15 environment.
- 8 GB RAM minimum; 16 GB recommended.
- GPU or Google Colab GPU is helpful for training, but not required to run the website.
- Internet access and Kaggle account/API credentials to download the dataset.

## 2. Install

Windows PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

If PowerShell blocks virtual-environment activation, use Command Prompt:

```bat
.venv\Scripts\activate.bat
```

## 3. Download the PlantVillage dataset

1. Create a Kaggle API token from your Kaggle account settings.
2. Place `kaggle.json` in `%USERPROFILE%\.kaggle\kaggle.json` on Windows, or `~/.kaggle/kaggle.json` on Linux.
3. Run:

```bash
python download_dataset.py
```

The expected final directory is `data/PlantVillage/<class_name>/<image files>`. Inspect the output and confirm the 38 class folders exist.

## 4. Train a model

Train MobileNetV2:

```bash
python train.py --model transfer --epochs 20 --batch-size 32
```

Train the custom CNN baseline as a separate run:

```bash
python train.py --model cnn --epochs 20 --batch-size 32
```

The app's model path is `saved_model/plant_disease_model.h5`. Each training run replaces that path, so copy/rename the model after each run if you want to preserve both models for comparison. Training writes a validation metrics JSON and confusion-matrix CSV into `saved_model/`. These are validation results, not independent test-set results.

## 5. Run the website

```bash
python app.py
```

Open `http://127.0.0.1:5000`.

- If no trained model exists, the website still opens and shows a setup notice.
- Prediction requests return a clear message until a trained model is available.
- After training, restart the Flask server so it loads the model.
- For a local demo, do not expose the development server directly to the public internet.

## API

`POST /api/predict` — send `multipart/form-data` with an `image` file field.

`GET /api/classes` — list configured classes.

`GET /health` — check whether the trained model loaded.

## Tests

```bash
pytest tests/ -v
```

TensorFlow must be installed to run preprocessing/model-related tests. If you encounter an import or version error, use Python 3.10 and the versions in `requirements.txt`.

## Docker

Build and run:

```bash
docker compose up --build
```

The trained model and dataset are not bundled in the image by default. Follow the dataset and training steps first if you want real predictions.

## Limitations

- PlantVillage images are often captured in controlled conditions. Real field images may produce lower accuracy.
- A high softmax score is not a calibrated probability that a diagnosis is correct.
- There is no dedicated, validated non-leaf classifier in this MVP.
- Field-image collection and testing still need to be done and documented.
- Treatment information is general guidance. Consult a qualified agricultural expert before taking important crop-protection decisions.
- Never report a target accuracy as a measured result; use the generated metrics from your actual run.

## Reference

Original reference repository: https://github.com/shahils5805s/Plant-Disease-Detection

Follow the license and attribution requirements in `LICENSE`.

## Team work distribution

See [`TEAM_WORK_DISTRIBUTION.md`](TEAM_WORK_DISTRIBUTION.md) for the full assignment based on the five members in the synopsis. Four members have similarly sized technical work areas (data/preprocessing, model training, evaluation/Grad-CAM, and Flask/frontend integration). The fifth member has a shorter role focused on the demo, presentation, screenshots, and documentation.

## Frontend-to-model connection

The website form in `templates/index.html` submits the uploaded image to Flask's `/predict` route. `app.py` validates the image, applies the same MobileNetV2 preprocessing used during training, calls the loaded Keras model, applies the confidence threshold, attempts Grad-CAM, looks up disease information, and renders `templates/result.html`. The JSON alternative is `POST /api/predict`.

The model is loaded once when `app.py` starts from `saved_model/plant_disease_model.h5`. This file is intentionally not included in the repository because it must be generated by training on your dataset. Run `python train.py --model transfer --epochs 20 --batch-size 32`, then restart the website. If the model file is missing, the site shows a setup notice rather than fake results.

A basic image-size and blur heuristic now runs before prediction. It is not a dedicated plant-leaf detector, so non-leaf detection remains a limitation and should not be described as fully implemented in the final report.
