# EFLIM
> A self-supervised, event-based deep learning method for FLIM under extreme low light.

## Introduction
EFLIM (Event-Based First-Photon FLIM) is a self-supervised deep learning method for fluorescence lifetime imaging microscopy (FLIM). Unlike conventional approaches that rely on photon histograms, EFLIM treats each excitation event as a binary process — either no photon is emitted, or a single first-arrival photon is detected with precise timing. By leveraging spatial and temporal context, EFLIM estimates lifetime under extremely low-light conditions (PPP<1), reducing photon demand by about three orders of magnitude. This enables fast, high-fidelity intravital imaging with strong robustness to intensity fluctuations. EFLIM opens new opportunities for studying dynamic biological processes in deep tissue.

## Overview
- **Event-based representation**: avoids histogram construction and directly models each excitation event.  
- **Self-supervised learning**: robust training without the need for paired datasets or ground-truth lifetimes.  
- **Extreme low light capability**: accurate lifetime estimation even below 1 photon per pixel (PPP).
- **Robustness to artifacts**: stable performance despite photobleaching, motion, or intensity fluctuations.  
- **Broad applicability**: enables fast, minimally invasive imaging of dynamic molecular processes in vivo.

## Workflow

### 1. Photon-arrival dataset preparation
- Rearrange raw photon data into **a sequence of frames that contain the arrival times of all photons**.
- For **Becker & Hickl** systems: convert `.SPC` files into `.tif` format.  
- For **PicoQuant** systems: convert `.PTU` files into `.tif` format.  
- We recommend using **at least 500 frames** with **PPP > 0.1** in regions of interest.  

### 1.1. Simulation (if raw data are unavailable)
If you do not have experimental raw data, you can generate synthetic photon arrivals using the provided MATLAB scripts.  

Example:  
```
./0_simulations/run_simu_USAF1951.m
```

This script simulates **500 frames** with **PPP = 0.5** and saves the dataset to:
```
./simu_USAF1951_PPP0.5
```

Inside this folder, you will find:
- **Photon-arrival frames** : ./simu_USAF1951_PPP0.5/raw/frame*.tif  
- **Ground truth**: ./simu_USAF1951_PPP0.5/lt_gt/lt_gt.tif
- **FastFLIM** (center-of-mass method, CMM):  ./simu_USAF1951_PPP0.5/lt_fastflim/lt_fastflim.tif
- **Intensity-weighted lifetime visualizations**, saved with the default [colormap](https://uigradients.com/) suffix `_weddingdayblues` (e.g., ./simu_USAF1951_PPP0.5/lt_fastflim/lt_fastflim_lt500-3500_in0-0.5_weddingdayblue.tif)

### 1.2. Experimental dataset
We provide a representative raw SPC dataset (Becker&Hickl, HPM-100-07, SPC-QC-104) for reproducibility and testing purposes.

The raw SPC file can be downloaded from:
```
https://drive.google.com/file/d/1cnEcXbqJvVVJ8ZdCexfEFmzAmhLR2nbM/view?usp=drive_link
```

After downloading, place the .SPC file into:
```
./spc2tiff/ExampleData/
```

Then run the MATLAB script:
```
./spc2tiff/spc2tiff.m
```

This script converts the raw SPC photon stream into a sequence of TIFF frames that can be directly used for EFLIM training and inference.

The converted TIFF files will be saved to:
```
./spc2tiff/output/
```

### 2. Python training and inference
Once the dataset is prepared, you can train and evaluate EFLIM using the provided Python code.

#### Requirements
- **Python ≥ 3.9**  
- **GPU support** (CUDA-enabled GPU recommended)  
- **Additional Python packages**:  
  - numpy  
  - scipy  
  - tifffile  
  - tqdm  
  - matplotlib  

### Create a new conda environment (recommended):
```bash
conda create -n eflim python=3.10 -y
conda activate eflim
```

### GPU support
To run EFLIM efficiently on a GPU, make sure you have a working CUDA toolkit installed.
The recommended way is to install PyTorch together with the matching CUDA version directly from the [official PyTorch website](https://pytorch.org/get-started/locally/).

For example, on a machine with **CUDA 11.8**, you can install PyTorch with:
```bash
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu118
```

### Install dependencies:
```bash
pip install -r requirements.txt
```

### Training example:
```bash
python run_EFLIM.py \
  --folderName .//simu_USAF1951_PPP0.5//raw
```

If you want to train on a specific GPU (e.g., GPU 2):
```bash
CUDA_VISIBLE_DEVICES=2 \
python run_EFLIM.py \
  --folderName .//simu_USAF1951_PPP0.5//raw
```

This script will:
- Load the raw data
- Perform training and inference for both lifetime and intensity
- Output a **lifetime video**, an **intensity video** and an **intensity-weighted lifetime visualization**