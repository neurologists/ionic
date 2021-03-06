Salt Deposit Identification in Python using Tensorflow
==================

© 2018 Stephen Vondenstein & Matthew Buckley

## About This Program

This program identifies salt deposits by analyzing subsurface images. The network design is based on the FC-DenseNet103 network, outlined in the [One Hundred Layers Tiramisu](https://arxiv.org/pdf/1611.09326.pdf) paper. Additional optimizations are made to process depth data and improve masks, which are outlined below.

This implementation achieved an accuracy of 83% (0.832 mean IoU) on the Kaggle public leaderboard. 

## Table of Contents

[Data](#data)

[Model](#model)

[Usage](#usage)
* [Setup](#setup)
* [Training](#training)
* [Inference](#inference)
* [Submitting to Kaggle](#computing-rle--assembling-submission-for-kaggle)

[Argument List](#arguments)

[File Structure](#file-structure)

[Licensing & Resources](#licensing--resources)

## Data

This project was trained and tested using the dataset from the [TGS Salt Identification Challenge](https://www.kaggle.com/c/tgs-salt-identification-challenge) Kaggle competition. (If you'd like to use this dataset for commercial purposes, you may have to contact the sponsor of the competition).

The source images and masks in this dataset are greyscale images of size 101 by 101, and the depth information for each image is included in a **`.csv`** file. Preprocessing is done to resize the images to achieve a usable tensor shape in the model. Postprocessing is applied to ensure a smooth mask, and to compute the RLE (run length encoded) masks, which is the encoding required by the competition sponsor.

## Model

The model is based closely on the [FC-DenseNet103](https://github.com/SimJeg/FC-DenseNet) and accompanying [paper](https://arxiv.org/pdf/1611.09326.pdf). Integrating the depth data from the Kaggle dataset was done by concatenating a 'depth tensor' (a tensor holding the depth data) with the bottleneck layer of the network.

## Usage

### Setup

1. Clone locally via GitHub Desktop or CLI: `git clone https://github.com/KaggleBuoys/ionic.git` and follow the directions below to train and infer.
2. Navigate to the **`data/`** directory and run the download script to download the dataset: `./download.sh` -- If you plan to use your own dataset instead, ensure that the file structure of the **`data/`** directory matches the one in the [file structure](#file-structure) section.
3. Ensure that the required dependencies are installed: `pip install -r requirements.txt`

NOTE: It is possible that this dataset will become unavailable for download at some time after the competition closes. If this happens, the download script will no longer function and you will have to supply your own data to train and infer using this model.

### Training

Run **`main.py`** with the `-t` argument: `python main.py -t`

NOTE: The training portion of this program is set to use a default batch size of 16 images. The network was trained using a computer with an Nvidia Tesla V100 GPU, which has 16GB of VRAM. If you are training this network using a GPU with less VRAM, you may need to reduce the batch size to avoid OOM errors. For more information about using different training parameters, including reducing the batch size, please consult the [arguments](#arguments) section of the README.

### Inference

Run **`main.py`** with the `-i` argument: `python main.py -i`

NOTE: To run inference, at least one saved model must be available to use. If none is available, you should first train the model for at least one epoch. To use different inference parameters, please consult the [arguments](#arguments) section of the README.

### Computing RLE & Assembling Submission for Kaggle

Run **`main.py`** with the `-r` argument: `python main.py -r`

NOTE: To assemble a submission, valid predictions must be available to use. If none are available, you should first run inference to generate at least one prediction. To use different submission parameters, please consult the [arguments](#arguments) section of the README.

## Arguments

### Definitions

The following is a list of arguments with definitions and default values for use when running the program from **`main.py`**:

| Argument | Definition | Default Value |
| --- | --- | --- |
| `-a`, `--augment` | When present, data is augmented to increase the size and variation of the training set | _false_ |
| `-b`, `--batch_size` | Batch size for training | **16** |
| `-c`, `--classes` | Number of distinct classes for inference | **2** |
| `-ci`, `--compute_iou` | Compute IoU of a submission against train.csv | _false_ |
| `-d`, `--data_path` | Path to the directory containing test and training data | **`data/`** |
| `-dp`, `--dropout_percentage` | Dropout percentage for dropout layer| **0.2** |
| `-e`, `--epochs` | Number of epochs to train | **32** |
| `-i`, `--infer` | When present, the program will run inference | _false_ |
| `-k`, `--growth_k` | Growth rate for Tiramisu | **16** |
| `-l`, `--log_path` | Path to the directory to save tensorboard logs | **`models/tensorboard/`** |
| `-lr`, `--learning_rate` | Learning rate for the optimizer | **1e-3** |
| `-m`, `--model_path` | Path to the directory in which to save trained models | **`checkpoints/`** |
| `-mk`, `--max_to_keep` | Maximum number of epoch checkpoints to save -- to keep all checkpoints, use 0 | **5** |
| `-o`, `--optimize` | Use the optimizer to perform hyperparameter optimization | _false_ |
| `-op`, `--optimizer_path` | Path to the directory in which to save optimizer data | **`optimizer/`** |
| `-p`, `--prediction_path` | Path to the directory in which to save predictions | **`predictions/`** |
| `-r`, `--rle` | When present, the program will compute the rle and prepare a submission | _false_ |
| `-s`, `--submission_path` | Path to the directory in which to save submissions | **`submissions/`** |
| `-t`, `--train` | When present, the program will train the model | _false_ |
| `-v`, `--validation_split` | Percentage of the training set to use as the validation set | **0.25** |
| `-x`, `--rle_to_image` | Convert .csv submissions containing RLE data to .png images | _false_ |

### Compatibility

While the `-t`, `-i` and `-r` arguments will function properly when called individually, if multiple arguments are present, the program **should** first train the network, then run inference on the trained model and prepare a submission from the generated predictions. This is currently being tested and may not function properly at this time.

## File Structure

In order to give an easier understanding of the project's design and execution, the file structure is explained below. Files and directories denoted with  `*` are not included, but required for the program to run properly. Those denoted with `**` are generated by the program when executed, assuming default arguments.

```
.
├── ** checkpoints                   -  Directory containing saved models.
│
├── data
│   ├── download.sh                  -  Script used to download the dataset.
│   ├── * depths.csv                 -  File containing depth data for each image.
│   ├── * train.csv                  -  File containing ids and RLE masks for training data.
│   ├── * test
│   │     └── * images
│   │           └── * <images>.png   -  The set of images for which to make predictions.
│   └── * train
│         ├── * images
│         │     └── * <images>.png   -  The set of images to train the network.
│         └── * masks
│               └── * <images>.png   -  The set of masks that corresponds to the training images.
│ 
├── docs
│   ├── LICENSE                      -  License pertaining to this repository.
│   └── README.md                    -  This document.
│
├── ** optimizer                     -  Directory containing saved optimizer data.
│
├── src
│   ├── agents
│   │   ├── optimizers
│   │   │   └── tiramisu_hyper.py    -  Hyperparameter optimizer.
│   │   ├── baseagent.py             -  Base class for predicter and trainer
│   │   ├── predicter.py             -  Helper file to run inference with the model.
│   │   └── trainer.py               -  Helper file to train the model. 
│   ├── helpers
│   │   ├── data_generator.py        -  Iterator for dataset.
│   │   ├── postprocess.py           -  Operations to clean up and resize the predicted masks.
│   │   └── preprocess.py            -  Operations to prepare input data for the network.
│   ├── models
│   │   ├── layers.py                -  Definitions for various layers within the network.
│   │   └── tiramisu.py              -  The network architecture.
│   └── utils
│       ├── logger.py                -  Functions to enable tensorboard.
│       ├── metrics.py               -  Functions to compute loss and accuracy.
│       ├── parser.py                -  The parser for parsing arguments to model configuration.
│       ├── rle.py                   -  Function to compute the RLE of the predicted masks.
│       └── utility.py               -  Other utilities set up the environment for the program.
│   
├── predictions
│   └── ** <predictions>.png         -  Predicted masks for the test set.
│
├── submissions
│   ├── sample_submission.csv        -  Template for Kaggle submissions provided by Kaggle.
│   └── ** submission-XX.csv         -  Submission files ready for upload to Kaggle.
│
├── tests
│   ├── iotest.py                    -  Script to test preprocessing and postprocessing code.
│   ├── iou_test.py                  -  Script to test iou calculation and compare validation data.
│   └── rle_test.py                  -  Script to test the RLE calculation and convert RLE-encoded images to .png files.
│ 
├── main.py                          -  The main entry point for the program.
│ 
└── requirements.txt                 -  File containing list of python dependencies
```

## Licensing & Resources

The model design is based on the [One Hundred Layers Tiramisu](https://arxiv.org/pdf/1611.09326.pdf) paper. The creators of the paper also provide a GitHub repository, [FC-DenseNet](https://github.com/SimJeg/FC-DenseNet), which helped us conceptualize the implementation and better understand how it functions. The [Fully Convolutional DenseNet Tensorflow](https://github.com/HasnainRaz/FC-DenseNet-TensorFlow) implementation by GitHub user [HasnainRaz](https://github.com/HasnainRaz) was used as a reference for the Tensorflow implementation of the model. While our end result differed a good bit, this implementation was helpful in designing the network used in this project and is a great reference or starting point for using Tiramisu in Tensorflow.

[MrGemy95](https://github.com/MrGemy95)'s [Tensorflow Project Template](https://github.com/MrGemy95/Tensorflow-Project-Template) is another invaluable resource for starting new Tensorflow projects. Although some of the template code was modified before being used in our project, being able to review the project structure provided a lot of insight.

The [Hyper Engine](https://github.com/maxim5/hyper-engine) optimizer was used for hyperparameter tuning on this project.
