# Statewide Visual Geolocalization in the Wild (ECCV 2024)

#### [Florian Fervers](https://fferflo.github.io/), [Sebastian Bullinger](https://sbcv.github.io/), [Christoph Bodensteiner](https://scholar.google.de/citations?user=eQS65kAAAAAJ), [Michael Arens](https://scholar.google.de/citations?user=Sg5ZkXwAAAAJ), [Rainer Stiefelhagen](https://cvhci.anthropomatik.kit.edu/people_596.php)

### Links: [Paper](https://arxiv.org/abs/2409.16763) | [Poster](https://fferflo.github.io/assets/img/statewide24-poster.jpg) | [Examples](https://photos.app.goo.gl/xZYcsvSQg7vq83V68)

*The poster will be presented at ECCV on Friday, October 4th, 10:30-12:30 at poster board #139.*

---

![summary](https://github.com/fferflo/statewide-visual-geolocalization/blob/main/images/summary.jpg)

## Overview

1. [Installation](#installation)
2. [Dataset](#dataset)
3. [Training](#training)
4. [Evaluation](#evaluation) (includes pretrained weights)
5. [Examples](https://photos.app.goo.gl/xZYcsvSQg7vq83V68): A photo album that contains >2000 randomly chosen street-view images and corresponding predictions from our model.

## Installation

1.  Install Jax with GPU support: https://jax.readthedocs.io/en/latest/installation.html
2.  Clone this repository:
    ```bash
    git clone https://github.com/fferflo/statewide-visual-geolocalization
    cd statewide-visual-geolocalization
    ```
3.  Install the remaining dependencies:
    ```bash
    pip install -r requirements.txt
    ```

## Dataset

We train and evaluate our method on street-view images from the [Mapillary platform](https://www.mapillary.com/) and aerial imagery from [Massachusetts](https://www.mass.gov/orgs/massgis-bureau-of-geographic-information), [Washington DC](https://opendata.dc.gov/), [North Carolina](https://www.nconemap.gov/) (states in the US) and [Berlin-Brandenburg](https://data.geobasis-bb.de/), [NRW](https://www.opengeodata.nrw.de/produkte/), [Saxony](https://www.geodaten.sachsen.de/) (states in Germany).

Please follow [these instructions](dataset) to download the data.

## Training

1.  Fill in the dataset paths indicated by `TODO` in the configuration file `config/main.yaml`. The entries should look something like this:
    ```yaml
    train:
      list:
        - tiles-path: .../data/opennrw
        path: .../data/mapillary-opennrw

    test:
      path: .../data/mapillary-boston100km2
      tiles:
        - path: .../data/massgis/utm19
      geojson: .../data/boston100km2.geojson
    ```

2.  Run the training script:
    ```bash
    python3 train.py --output .../train --config config/main.yaml
    ```
    The results will be stored in `.../train-YYYY-MM-DDTHH-mm-ss`. The training uses all available GPUs by default.

## Evaluation

1.  Create a reference database for a search region by running the following script:
    ```bash
    python create_reference_database.py --train .../train-YYYY-MM-DDTHH-mm-ss --output .../refdb-massgis --tiles .../data/massgis/utm19 .../data/massgis/utm18
    ```
    This will create a division of the region into cells, predict embeddings for all cells, create a FAISS index for efficient retrieval and store everything in the output directory. *This might take several days depending on your hardware setup and search region size.*

    By default, the search region is defined to cover all tiles that are specified in `--tiles`. The argument accepts multiple tile datasets, such as the overlapping UTM18 and UTM19 regions of Massachusetts. Optionally, a geojson file can be passed to the script via `--geojson` to define a custom search region as a subset of the region covered by the tiles.

    The output folder will contain the files:
    ```
    aerial_features.bin         # Embeddings for all cells
    cellregion.npz              # Division of the region into cells
    faiss.index                 # FAISS index that can be loaded via faiss.read_index("faiss.index")
    config.yaml                 # Configuration parameters of the search region, model, etc
    model_weights.safetensors   # Model weights used to create the embeddings
    ``` 

2.  Localize query images against the reference database by running the following script:
    ```bash
    python localize.py --query .../data/mapillary-boston100km2 --reference .../refdb-massgis --stride 1
    ```
    This will predict embeddings for all street-view photos in the given dataset, and localize them against the reference database. The `--stride` parameter can be used to localize only a subset of the images (e.g. every 10th image with `--stride 10`).

    The script will print the recall of the localization:
    ```
    Recall@1<50m: 60.0%
    ```

## Issues

Feel free to open an issue in this Github repository if you have any problems with the code or data.