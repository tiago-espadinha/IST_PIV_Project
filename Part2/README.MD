# Part 2: Camera Calibration & Depth Map Reconstruction

## Team Members

- Diogo Rosa, ist93044
- Tiago Simões, ist96329
- Tomás Santos, ist100099
- Filipe Tendeiro, ist108325

## Project Description

This module performs stereo camera calibration using chessboard patterns and reconstructs depth maps from stereo image pairs using two complementary approaches:

1. **Neural Network Approach** (`compute_transform_NN.py`): Uses a pre-trained CREStereo model (ONNX format) to estimate disparity directly from stereo pairs, then computes depth and 3D point clouds using calibration data.

2. **Traditional Stereo Approach** (`compute_transform_STEREO.py`): Uses SIFT feature matching, fundamental matrix estimation, stereo rectification, and OpenCV's StereoSGBM with WLS filtering to compute disparity maps, then derives depth and point clouds.

Both methods output disparity maps, depth maps, and colored 3D point clouds (Open3D .pcd format).

## Requirements

- configparser
- dataclasses
- glob
- matplotlib
- numpy
- onnxruntime
- open3d
- opencv-python (cv2)
- opencv-contrib-python (for ximgproc module)
- os
- scipy
- time

### Data Requirements

#### Camera Calibration Input

Place chessboard images in:

- `calib_parameters/input/images_L/*.jpeg` - left camera views
- `calib_parameters/input/images_R/*.jpeg` - right camera views

**Chessboard specifications** (hardcoded in `camera_calibration.py`):

- Inner corners: 9×6
- Square size: 25 mm
- Image resize: 600×800 pixels

#### Stereo Dataset Input

Place stereo pairs in:

- `my_datasets/dataset1/left.jpeg` - left image
- `my_datasets/dataset1/right.jpeg` - right image

#### Pre-trained Models

Located in `models/` (also copied to `pre_trained_models/`):

- `crestereo_combined_iter10_720x1280.onnx` (default, ~24 MB)
- Additional variants in `models/other_models/` (smaller resolutions)

## Installation

1. Install dependencies as specified above:

```bash
pip install numpy scipy matplotlib opencv-python opencv-contrib-python onnxruntime open3d
```

1. Prepare data files as required
2. Configure any necessary settings

## Usage

### Configuration

Edit `conf_file.cfg`:

| Parameter            | Description              | Default                                                      |
| -------------------- | ------------------------ | ------------------------------------------------------------ |
| `model_path`         | ONNX model path          | `pre_trained_models/crestereo_combined_iter10_720x1280.onnx` |
| `calib_path`         | Calibration data root    | `calib_parameters/`                                          |
| `dataset_path`       | Stereo dataset path      | `my_datasets/dataset1/`                                      |
| `left_image_name`    | Left image filename      | `left.jpeg`                                                  |
| `right_image_name`   | Right image filename     | `right.jpeg`                                                 |
| `output_path_NN`     | NN results directory     | `output/NN/`                                                 |
| `output_path_STEREO` | Stereo results directory | `output/STEREO/`                                             |

### Running the Project

#### 1. Camera Calibration (`camera_calibration.py`)

**Must run first!** Generates calibration matrices required by depth scripts.

```bash
python camera_calibration.py [config_file.cfg]
```

Process:

1. Loads chessboard images from `calib_path/input/images_L/` and `images_R/`
2. Detects corners, refines with sub-pixel accuracy
3. Computes camera matrix, distortion coefficients, rotation/translation vectors
4. Saves `calib_L.mat` and `calib_R.mat` to `calib_path/output/`

Outputs (MATLAB .mat format):

- `cameraMatrix` - 3×3 intrinsic matrix
- `distCoeff` - distortion coefficients
- `rvecs`, `tvecs` - rotation/translation vectors per image
- `objpoints`, `imgpoints` - 3D/2D point correspondences
- Reprojection error printed to console

#### 2a. Depth Map via Neural Network (`compute_transform_NN.py`)

Uses CREStereo ONNX model for disparity estimation.

```bash
python compute_transform_NN.py [config_file.cfg]
```

Requirements:

- `stereo_model.py` must be in same directory
- ONNX model at `model_path`
- Calibration matrices from step 1

Pipeline:

1. Loads and resizes stereo pair to 600×800
2. Computes SIFT matches for fundamental matrix estimation
3. Runs CREStereo model to get disparity map
4. Computes depth: `depth = (fx × ‖t‖) / disparity`
5. Generates 3D point cloud with Open3D

Outputs to `output_path_NN/`:

- `Disparity_Map.jpeg` - colorized disparity (model output)
- `Disparity_Map2.jpeg` - raw disparity with colorbar
- `Depth_Map.jpeg` - normalized depth map
- `Point_Cloud.pcd` - 3D point cloud (Open3D format)

#### 2b. Depth Map via Traditional Stereo (`compute_transform_STEREO.py`)

Uses SIFT + StereoSGBM + WLS filtering - no neural network required.

```bash
python compute_transform_STEREO.py [config_file.cfg]
```

Requirements:

- Calibration matrices from step 1
- OpenCV contrib modules (ximgproc)

Pipeline:

1. Loads grayscale stereo pair, resizes to 600×800
2. SIFT detection + FLANN matching with ratio test (0.5)
3. Fundamental matrix (FM_LMEDS) → Essential matrix → R, t recovery
4. Stereo rectification (cv2.stereoRectify)
5. StereoSGBM disparity + right matcher + WLS filter (λ=8000, σ=1.5)
6. Interpolation of invalid (zero) disparities
7. Depth computation: `depth = (fx × ‖t‖) / filtered_disparity`
8. Point cloud generation

Outputs to `output_path_STEREO/`:

- `Matched_SIFT.jpeg` - keypoint matches visualization
- `Epipolar_Lines.jpeg` - epipolar geometry verification
- `Rectified_Images.jpeg` - rectified stereo pair
- `Disparity_Map.jpeg` - filtered disparity map
- `Depth_Map.jpeg` - computed depth map
- `Point_Cloud.pcd` - 3D point cloud (Open3D format)

## Outputs

### Calibration Outputs (`calib_parameters/output/`)

- `calib_L.mat` - Left camera calibration data
- `calib_R.mat` - Right camera calibration data

### Neural Network Outputs (`output/NN/`)

- `Disparity_Map.jpeg` - CREStereo colorized disparity
- `Disparity_Map2.jpeg` - Raw disparity with matplotlib colorbar
- `Depth_Map.jpeg` - Normalized depth visualization
- `Point_Cloud.pcd` - Colored point cloud (viewable in Open3D, CloudCompare, MeshLab)

### Stereo Matching Outputs (`output/STEREO/`)

- `Matched_SIFT.jpeg` - SIFT correspondences
- `Epipolar_Lines.jpeg` - Epipolar lines on rectified images
- `Rectified_Images.jpeg` - Rectified left/right images
- `Disparity_Map.jpeg` - WLS-filtered disparity
- `Depth_Map.jpeg` - Depth map visualization
- `Point_Cloud.pcd` - Colored point cloud

## Notes

- **Calibration is mandatory** before running either depth script
- **Neural network approach** requires `stereo_model.py` + ONNX model (~24 MB download if not present)
- **Stereo approach** requires `opencv-contrib-python` for `cv2.ximgproc` (WLS filter)
- Both scripts resize input images to 600×800 - modify `frameSize` in code for other resolutions
- Depth scale uses `‖t‖` (baseline) from essential matrix decomposition
- Point clouds use intrinsic matrix from left camera; transform flips Y/Z for visualization
- SIFT ratio test threshold: 0.5 (Lowe's ratio test)
- StereoSGBM: window_size=3, numDisparities=80 (5×16), minDisparity=16
- WLS filter: λ=8000, σ=1.5 (balance smoothness vs. edge preservation)
