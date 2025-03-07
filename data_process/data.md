# Data Preparation Guide for ScanNet and ScanNet++

## Table of Contents
- [Data Preparation Guide for ScanNet and ScanNet++](#data-preparation-guide-for-scannet-and-scannet)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Prerequisites](#prerequisites)
  - [ScanNet Data Preparation](#scannet-data-preparation)
    - [1. Download ScanNet Data](#1-download-scannet-data)
    - [2. Extract .sens Files](#2-extract-sens-files)
    - [3. Data Processing](#3-data-processing)
    - [4. Data Structure](#4-data-structure)
  - [Common Issues](#common-issues)
    - [1. Download Issues](#1-download-issues)
    - [2. Processing Issues](#2-processing-issues)
    - [3. Troubleshooting](#3-troubleshooting)
  - [Additional Notes](#additional-notes)

## Overview
This document provides instructions for preparing ScanNet and ScanNet++ datasets for training and evaluation.

## Prerequisites
- Python 3.7+
- Sufficient storage space (>1TB recommended)
- Required Python packages:
  ```bash
  pip install -r data_process/requirements.txt
  ```

## ScanNet Data Preparation

### 1. Download ScanNet Data
1. Visit the [official ScanNet repository](https://github.com/ScanNet/ScanNet)
2. Fill out the [Terms of Use agreement](http://kaldir.vc.in.tum.de/scannet/ScanNet_TOS.pdf)
3. Send the signed agreement to scannet@googlegroups.com
4. You will receive download instructions and access credentials via email

Example download command (after receiving credentials):
```bash
# Download ScanNet v2 data
python download-scannet.py -o ./data/scannet --type .sens
```

### 2. Extract .sens Files
We provide a parallel processing script to efficiently extract data from .sens files:

```bash
# Run the parallel export script
python data_process/scannet/export_data/export.py \
    --input_dir ./data/scannet \
    --output_dir ./data/scannet_extracted \
    --num_workers 8
```

The export script provides the following features:
- Parallel processing using multiple CPU cores
- Automatic skipping of already processed scenes
- Progress tracking with tqdm

Arguments:
- `--input_dir`: Directory containing the ScanNet dataset
- `--output_dir`: Directory to save extracted data
- `--num_workers`: Number of parallel workers (default: half of CPU cores)

Output structure for each scene:
```
data/scannet_extracted/
├── scene0000_00/
│ ├── color/ # RGB images in jpg format
│ │ ├── 0.jpg
│ │ ├── 1.jpg
│ │ └── ...
│ ├── depth/ # Depth images in png format (16-bit, depth shift 1000)
│ │ ├── 0.png
│ │ ├── 1.png
│ │ └── ...
│ ├── pose/ # Camera poses (4x4 matrices)
│ │ ├── 0.txt
│ │ ├── 1.txt
│ │ └── ...
│ └── intrinsic/ # Camera parameters
│ ├── intrinsic_color.txt # Color camera intrinsics
│ ├── intrinsic_depth.txt # Depth camera intrinsics
│ ├── extrinsic_color.txt # Color camera extrinsics
│ └── extrinsic_depth.txt # Depth camera extrinsics
├── scene0000_01/
│ ├── color/
│ ├── depth/
│ ├── pose/
│ └── intrinsic/
└── ...
```

### 3. Data Processing
```bash
# Process raw data
python -m data_process.scannet.scannet_processor \
    --root_dir data/scannet_extracted \
    --save_dir data/scannet_processed \
    --device cuda \
    --num_workers 8
```

Arguments:
- `--root_dir`: Path to the extracted ScanNet data directory (default: "data/scannet_extracted")
  - Should contain the extracted .sens files organized by scene
  - Each scene should have color/, depth/, pose/, and intrinsic/ subdirectories

- `--save_dir`: Path where processed data will be saved (default: "data/scannet_processed")
  - Will create if directory doesn't exist
  - Processed data will be organized by scene with standardized format

- `--device`: Computing device to use (default: "cuda")
  - "cuda": Use GPU acceleration (recommended)
  - "cpu": Use CPU only (slower)

- `--num_workers`: Number of parallel processing workers (default: 8)
  - Higher values may speed up processing but use more memory
  - Recommended: Set to number of CPU cores or less

Note: Ensure sufficient disk space in save_dir (>500GB recommended for full dataset)

### 4. Data Structure
After processing, the ScanNet data will be organized in the 
following structure:
```
data/scannet_processed/
├── scene0000_00/
│   ├── color/         # Directory containing RGB images
│   │   ├── 000000.png
│   │   └── ...        # Additional RGB images
│   ├── depth/         # Directory containing Depth maps
│   │   ├── 000000.png
│   │   └── ...        # Additional Depth maps
│   └── pose/          # Directory containing Camera poses
│       ├── 000000.npz
│       └── ...        # Additional camera pose files
├── scene0000_01/
└── ...                 # Additional scenes
```
## Common Issues

### 1. Download Issues
- Ensure you have registered and obtained proper access credentials
- Check network connection and proxy settings
- Verify sufficient disk space

### 2. Processing Issues
- Memory errors: Try processing with smaller batch sizes
- Missing files: Verify complete dataset download
- Label mapping errors: Check for updated label mapping files

### 3. Troubleshooting
If you encounter any issues:
1. Check the log files
2. Verify file permissions
3. Ensure all prerequisites are properly installed
4. Check system resources (RAM, disk space)

## Additional Notes
- Backup your raw data before processing
- Consider using symbolic links for large datasets
- Monitor system resources during processing
