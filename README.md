# Satellite Image Segmentation using U-Net (Dubai Dataset)

This project implements a U-Net model using Keras/TensorFlow for semantic segmentation of satellite imagery, specifically targeting the Dubai dataset structure provided. It segments images into 6 classes: Water, Land, Road, Building, Vegetation, and Unlabeled.




---

## Table of Contents

1.  [Introduction](#introduction)
2.  [Dataset](#dataset)
3.  [Features](#features)
4.  [Requirements](#requirements)
5.  [Installation](#installation)
6.  [Usage](#usage)
    *   [Data Preparation](#data-preparation)
    *   [Training & Evaluation](#training--evaluation)
    *   [Prediction Example](#prediction-example)
7.  [Model Architecture](#model-architecture)
8.  [Loss Function & Metrics](#loss-function--metrics)
9.  [Results](#results)
10. [Project Structure](#project-structure)
11. [Potential Improvements](#potential-improvements)
12. [License](#license)

---

## Introduction

This project focuses on performing semantic segmentation on satellite images. The goal is to assign a predefined class label to every pixel in an input image. The provided Python script handles data loading, preprocessing (patching, normalization, label conversion), model definition (U-Net), training, evaluation, and saving the final model. It uses the TensorFlow/Keras framework.

---

## Dataset

*   **Name:** Semantic Segmentation Dataset (Dubai Tiles) - *Assuming this based on the path structure; please update if a more official name exists.*
*   **Source:** The dataset location is expected to be defined by the `dataset_root` variable in the script (e.g., `D:\SIS\datasets\Semantic segmentation dataset`).
*   **Structure:** The dataset is expected to be organized into Tiles, with corresponding images and masks:
    ```
    <dataset_root>/
    ├── Tile 1/
    │   ├── images/
    │   │   ├── image_part_001.jpg
    │   │   └── ... (up to image_part_019.jpg)
    │   └── masks/
    │       ├── image_part_001.png
    │       └── ... (up to image_part_019.png)
    ├── Tile 2/
    │   ├── images/
    │   └── masks/
    ...
    └── Tile 7/
        ├── images/
        └── masks/
    ```
*   **Classes:** The masks use specific RGB color codes which are mapped to integer labels during preprocessing:
    *   `[ 34 169 226]` (Hex: `#22A9E2` - **Water**) -> Label `0`
    *   `[132  41 246]` (Hex: `#8429F6` - **Land**) -> Label `1`
    *   `[110 193 228]` (Hex: `#6EC1E4` - **Road**) -> Label `2`
    *   `[ 60  16 152]` (Hex: `#3C1098` - **Building**) -> Label `3`
    *   `[254 221  58]` (Hex: `#FEDD3A` - **Vegetation**) -> Label `4`
    *   `[155 155 155]` (Hex: `#9B9B9B` - **Unlabeled**) -> Label `5`

---

## Features

*   **Data Loading:** Reads JPG images and PNG masks from a tiled directory structure.
*   **Preprocessing:**
    *   Crops images/masks to dimensions divisible by the patch size.
    *   Splits images and masks into non-overlapping patches (default size: 256x256) using `patchify`.
    *   Normalizes image pixel values to the [0, 1] range using `MinMaxScaler`.
    *   Converts RGB color masks into single-channel integer labels (0-5).
    *   One-hot encodes labels for categorical loss functions.
*   **Model:** Implements a U-Net architecture using Keras functional API.
*   **Training:** Trains the model using `Adam` optimizer and a custom combined loss function. Splits data into 85% training and 15% validation sets.
*   **Loss Function:** Uses a combination of Weighted Dice Loss and Categorical Focal Loss (`total_loss`).
*   **Metrics:** Monitors `accuracy` and `jaccard_coef` (Intersection over Union - IoU).
*   **Evaluation:** Plots training/validation loss and IoU curves. Shows a visual comparison of a random test image, its ground truth mask, and the predicted mask.
*   **Model Saving:** Saves the trained model weights and architecture to an HDF5 file (`satellite_segmentation_full.h5`).

---

## Requirements

The script requires the following Python libraries:

*   `python >= 3.7`
*   `os` (built-in)
*   `scikit-learn`
*   `patchify`
*   `Pillow`
*   `numpy`
*   `opencv-python` (cv2)
*   `matplotlib`
*   `tensorflow >= 2.x` (includes `keras`)

---

## Installation

1.  **Clone the repository (if applicable):**
    ```bash
    git clone <your-repo-url>
    cd <your-repo-name>
    ```

2.  **Set up Dataset:** Ensure your Dubai dataset is available and the `dataset_root` variable in the script points to its correct location.

3.  **Create a virtual environment (recommended):**
    *   Using `venv`:
        ```bash
        python -m venv venv
        source venv/bin/activate  # On Windows: venv\Scripts\activate
        ```
    *   Using `conda`:
        ```bash
        conda create --name satseg_env python=3.9
        conda activate satseg_env
        ```

4.  **Install dependencies:** Create a `requirements.txt` file with the following content:
    ```txt
    scikit-learn
    patchify
    Pillow
    numpy
    opencv-python
    matplotlib
    tensorflow>=2.0 # Or specify your exact TF version
    ```
    Then install using pip:
    ```bash
    pip install -r requirements.txt
    ```
    *(Note: The original script includes `pip install` commands within the code, which is not standard practice. Using `requirements.txt` is preferred).*

---

## Usage

The provided Python script (`GeoSegmention.py`) handles most steps sequentially.

### Data Preparation

1.  **Modify `dataset_root`:** Open the script (`Geosegmentation.py`) and update the `dataset_root` variable to the correct path where your dataset resides.
    ```python
    dataset_root = r'D:\path\to\your\Semantic segmentation dataset' # <-- UPDATE THIS
    ```
2.  **Patch Size:** You can adjust the `image_patch_size` variable (default is `256`) if needed.

### Training & Evaluation

1.  **Run the Script:** Execute the Python script from your terminal within the activated virtual environment:
    ```bash
    Geosegmentation.py
    ```
2.  **Process:** The script will:
    *   Load images and masks.
    *   Preprocess the data (patching, scaling, label conversion).
    *   Split data into training and validation sets.
    *   Define and compile the U-Net model.
    *   Train the model for the specified number of epochs (default: 100). Training progress will be printed to the console.
    *   Plot and display the training/validation loss and IoU curves.
    *   Save the trained model as `satellite_segmentation_full.h5` in the same directory as the script.
    *   Display a visual comparison for a random image from the test set.

### Prediction Example

The last part of the script demonstrates how to load the saved model and predict on a single image from the test set. To predict on new, unseen images, you would need to:

1.  Load the saved model:
    ```python
    from tensorflow import keras
    model = keras.models.load_model(
        "satellite_segmentation_full.h5",
        custom_objects={
            "total_loss": total_loss, # Or your specific loss function reference
            "jaccard_coef": jaccard_coef # Or your specific metric reference
        }
    )
    ```
2.  Load the new image using `cv2`.
3.  Preprocess the new image exactly like the training data:
    *   Crop it to be divisible by `patch_size`.
    *   Patchify it.
    *   Scale each patch using `MinMaxScaler` (fit/transform *per patch* or use a scaler fitted on training data).
    *   Combine patches if predicting on the whole image or process patch-by-patch.
4.  Use `model.predict()` on the preprocessed input.
5.  Apply `np.argmax(prediction, axis=3)` to get the final class label for each pixel.
6.  Optionally, stitch predicted patches back together if the input image was large.

---

## Model Architecture

The project uses a U-Net architecture implemented in the `multi_unet_model` function.

*   **Input:** (256, 256, 3) image patches.
*   **Encoder:** Consists of repeating blocks of two 3x3 Convolutional layers (ReLU activation, 'he_normal' initializer, 'same' padding), followed by Dropout (0.2) and 2x2 Max Pooling. The number of filters increases down the path (16 -> 32 -> 64 -> 128).
*   **Bottleneck:** Two 3x3 Convolutional layers (256 filters) with ReLU and Dropout (0.2).
*   **Decoder:** Consists of repeating blocks of:
    *   2x2 Conv2DTranspose (Upsampling).
    *   Concatenation with the corresponding feature map from the encoder path.
    *   Two 3x3 Convolutional layers (ReLU, 'he_normal', 'same' padding) with Dropout (0.2). The number of filters decreases up the path (128 -> 64 -> 32 -> 16).
*   **Output Layer:** A final 1x1 Convolutional layer with `softmax` activation to produce probability maps for the 6 classes.

---

## Loss Function & Metrics

*   **Loss:** `total_loss` - A combination of:
    *   `dice_loss_weighted`: Dice loss calculated per class and weighted (weights currently hardcoded as equal: `[0.166, 0.166, 0.166, 0.166, 0.166, 0.166]`). Aims to handle class imbalance and maximize overlap.
    *   `categorical_focal_loss`: Focal loss (alpha=0.25, gamma=2.0) to down-weight easily classified pixels and focus training on hard examples.
*   **Metrics:**
    *   `accuracy`: Standard pixel-wise accuracy.
    *   `jaccard_coef`: Intersection over Union (IoU), a standard metric for segmentation measuring the overlap between predicted and ground truth masks.

---

## Results

The script outputs:

1.  Plots showing the training and validation loss and IoU (`jaccard_coef`) over epochs. These plots help visualize model convergence and potential overfitting.
2.  A final saved model file: `satellite_segmentation_full.h5`.
3.  A visual comparison of a randomly selected test image, its ground truth mask, and the model's prediction.

*(Consider adding the final test set accuracy and IoU values here after training is complete, which can be obtained via `model.evaluate(X_test, y_test)`)*

---

## Project Structure

The project currently consists mainly of a single Python script and the dataset.
