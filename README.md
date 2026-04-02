# galamsey-mapper

Deep learning pipeline for detecting and mapping small-scale mining (galamsey) from satellite imagery.

Developed at the **Centre for Remote Sensing and Geographic Information Services (CERSGIS)**, University of Ghana, Legon.

## Overview

The pipeline accepts a Planet NICFI satellite mosaic and manually digitised training polygons as inputs and produces a binary prediction map classifying each pixel as galamsey or non-galamsey. It uses a U-Net segmentation model with a ResNet34 encoder, trained on 4-channel (B, G, R, NIR) imagery at 4.77 m resolution.

The full methodology is documented in `gala_mapper_documentation.pdf`.

## Repository structure

```
galamsey-mapper/
├── README.md
├── documentation/
    ├──gala_mapper_documentation.pdf          # Technical methodology document
├── notebooks/
│   ├── 01_data_preparation.ipynb  # Stages 1–3: mosaic, mask, patches, spatial split
│   ├── 02_training.ipynb          # Stages 4–5: model training and evaluation
│   └── 03_prediction.ipynb        # Stages 6–7: tiled prediction and post-processing
```

## Pipeline stages

| Stage | Description | Notebook |
|-------|-------------|----------|
| 1. Mask creation | Rasterise training polygons (Class 0/1/255) | `01_data_preparation` |
| 2. Patch generation | Slice mosaic and mask into 256×256 patches | `01_data_preparation` |
| 3. Spatial splitting | Block-based train/val/test split (70/15/15) | `01_data_preparation` |
| 4. Model training | U-Net/ResNet34, BCE+Dice loss, 50 epochs | `02_training` |
| 5. Evaluation | Precision, Recall, F1, IoU on held-out test set | `02_training` |
| 6. Prediction | Tiled sliding window over full mosaic | `03_prediction` |
| 7. Post-processing | Nodata mask, NDVI filter, built-up mask, sieve | `03_prediction` |

## Requirements

The notebooks are designed to run on [Google Colab](https://colab.research.google.com/) with a T4 GPU runtime. Dependencies are installed within each notebook. The core packages are:

- PyTorch
- segmentation-models-pytorch
- rasterio
- geopandas
- albumentations
- scikit-image
- GDAL

## Data requirements

- **Satellite imagery:** Planet NICFI analysis-ready mosaics (4 bands, ~4.77 m resolution, GeoTIFF). Can be a single mosaic or multiple tiles — the notebooks handle both.
- **Training labels:** Polygon shapefile with an integer `Class` field. Class 1 = galamsey, Class 0 = hard negatives (spectrally similar non-mining features). Unlabelled areas are ignored during training.
- **Built-up mask** (optional): Binary raster of settlements and urban areas for post-processing. Can be sourced from GHSL, OpenStreetMap, or institutional datasets.

## Usage

1. **Prepare data** — Run `01_data_preparation.ipynb`. Point it at your imagery tiles and training shapefile. Outputs a `split.tar.gz` archive containing the spatially-split patch dataset.

2. **Train model** — Run `02_training.ipynb`. Loads the patch archive, trains for 50 epochs, saves the best model checkpoint to Drive.

3. **Predict and clean** — Run `03_prediction.ipynb`. Applies the trained model to the full mosaic using tiled prediction, then runs post-processing (nodata masking, adaptive NDVI filtering, built-up masking, GDAL sieve). Outputs raw and cleaned prediction rasters.

The prediction notebook is memory-safe — both prediction and post-processing use tiled/strip-based I/O and run within Colab's free-tier RAM limits on large AOIs.

## References

- Couttenier, M., Di Rollo, S., Inguere, L., Mohand, M. and Schmidt, L. (2022). Mapping artisanal and small-scale mines at large scale from space with deep learning. *PLOS ONE*, 17(9), e0267963.
- Ronneberger, O., Fischer, P. and Brox, T. (2015). U-Net: Convolutional networks for biomedical image segmentation. In *MICCAI*, pp. 234–241. Springer.
- He, K., Zhang, X., Ren, S. and Sun, J. (2016). Deep residual learning for image recognition. In *Proceedings of the IEEE CVPR*, pp. 770–778.

## License

This project is developed by CERSGIS, University of Ghana. Contact the repository maintainers for licensing and usage terms.
