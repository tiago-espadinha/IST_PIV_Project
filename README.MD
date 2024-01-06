# Image Processing and Vision Project

## Team Members

- Diogo Rosa, ist93044
- Tiago Simões, ist96329
- Tomás Santos, ist100099
- Filipe Tendeiro, ist108325

## Project Description

This project implements computer vision techniques for video processing, camera calibration, and depth map reconstruction. It is divided into two main parts:

**Part 1 - Video Processing & Homography Estimation**: Processes video files to extract frames, detects SIFT keypoints and descriptors, enables manual keypoint selection for reference mapping, and computes homographies between frames using RANSAC-based robust estimation.

**Part 2 - Camera Calibration & Depth Reconstruction**: Performs stereo camera calibration using chessboard patterns, then reconstructs depth maps from stereo image pairs using two approaches: (1) a pre-trained neural network (CREStereo) and (2) traditional stereo matching with SIFT features and OpenCV's StereoSGBM algorithm. Both approaches generate disparity maps, depth maps, and 3D point clouds.

## Repository Structure

```
Project_PIV/
├── Part1/                          # Video processing and homography estimation
│   ├── process_video.py            # Extract frames & compute SIFT features
│   ├── get_keypoints.py            # Manual keypoint selection tool
│   ├── compute_transform.py        # Compute homographies (RANSAC)
│   ├── conf_file.cfg               # Configuration file
│   └── README.md
├── Part2/                          # Camera calibration & depth reconstruction
│   ├── camera_calibration.py       # Stereo camera calibration
│   ├── compute_transform_NN.py     # Depth via CREStereo neural net
│   ├── compute_transform_STEREO.py # Depth via SIFT + StereoSGBM
│   ├── stereo_model.py             # CREStereo ONNX model wrapper
│   ├── conf_file.cfg               # Configuration file
│   ├── calib_parameters/           # Calibration input/output
│   ├── my_datasets/                # Stereo image datasets
│   ├── pre_trained_models/         # ONNX models
│   └── README.md
├── models/                         # Pre-trained ONNX models (CREStereo variants)
├── .gitignore
└── README.md
```

## Requirements

### Part 1

- configparser
- math
- matplotlib
- numpy
- opencv-python (cv2)
- os
- random
- scipy
- sys

### Part 2

- configparser
- dataclasses
- glob
- matplotlib
- numpy
- onnxruntime
- open3d
- opencv-python (cv2)
- opencv-contrib-python (for ximgproc)
- os
- scipy
- time

### Data Requirements

#### Part 1

Place video files in the project directory (or specify paths in `conf_file.cfg`):

- Video file (e.g., `trymefirst.mp4`) - input video to process

Expected output directories created automatically:

- `processed_videos/<video_name>/frames/` - extracted frames
- `features.mat` - SIFT keypoints and descriptors (MATLAB format)
- `homography.mat` - computed homography matrices (MATLAB format)

#### Part 2

##### Camera Calibration

Place chessboard calibration images in:

- `calib_parameters/input/images_L/*.jpeg` - left camera images
- `calib_parameters/input/images_R/*.jpeg` - right camera images

Chessboard specifications (hardcoded in `camera_calibration.py`):

- Chessboard size: 9x6 inner corners
- Square size: 25mm
- Frame size: 600x800 pixels

##### Depth Reconstruction

Place stereo image pairs in:

- `my_datasets/dataset1/left.jpeg` - left image
- `my_datasets/dataset1/right.jpeg` - right image

Pre-trained models (in `models/` directory):

- `crestereo_combined_iter10_720x1280.onnx` (default, ~24MB)
- Additional variants in `models/other_models/`

## Installation

1. Install dependencies as specified above:

```bash
# Core dependencies
pip install numpy scipy matplotlib opencv-python opencv-contrib-python configparser

# Part 2 additional dependencies
pip install onnxruntime open3d
```

2. Prepare data files as required
3. Configure any necessary settings

## Usage

### Part 1: Video Processing & Homography

#### Configuration

Edit `Part1/conf_file.cfg` to set:

- `videos` - path to input video file
- `frames_directory` - output directory for extracted frames
- `image_map` - name of reference map image (first frame)
- `undersampling_factor` - frame sampling rate (e.g., 30 = every 30th frame)
- `keypoints_out` - output filename for SIFT features
- `transforms_out` - output filename for homographies
- `pts_in_map` / `pts_in_frame` - keypoint coordinates (set via `get_keypoints.py`)
- `transforms` - mode: `homography map` or `homography all`

#### Running the Project

```bash
cd Part1

# 1. Process video: extract frames and compute SIFT features
python process_video.py [config_file.cfg]

# 2. (Optional) Select reference keypoints manually
python get_keypoints.py [config_file.cfg]
# Click ≥4 corresponding points in map.jpg and frame_0030.jpg

# 3. Compute homographies
python compute_transform.py [config_file.cfg]
# Modes: 'map' (frame→map) or 'all' (all frame pairs)
```

### Part 2: Camera Calibration & Depth Reconstruction

#### Configuration

Edit `Part2/conf_file.cfg` to set:

- `calib_path` - path to calibration data (default: `calib_parameters/`)
- `dataset_path` - path to stereo image dataset (default: `my_datasets/dataset1/`)
- `left_image_name` / `right_image_name` - stereo pair filenames
- `model_path` - path to ONNX model (default: `pre_trained_models/crestereo_combined_iter10_720x1280.onnx`)
- `output_path_NN` - output directory for neural network results
- `output_path_STEREO` - output directory for stereo matching results

#### Running the Project

```bash
cd Part2

# 1. Camera Calibration (run first!)
python camera_calibration.py [config_file.cfg]
# Generates calib_L.mat and calib_R.mat in calib_parameters/output/

# 2a. Depth Map via Neural Network (CREStereo)
python compute_transform_NN.py [config_file.cfg]
# Requires: stereo_model.py and ONNX model

# 2b. Depth Map via Traditional Stereo (SIFT + StereoSGBM)
python compute_transform_STEREO.py [config_file.cfg]
# No neural network required
```

## Outputs

### Part 1

- **Frames**: `processed_videos/<name>/frames/frame_XXXX.jpg` + `map.jpg`
- **Features**: `features.mat` - MATLAB struct with keypoints & descriptors per frame
- **Homographies**: `homography.mat` - MATLAB struct with transformation matrices
  - `map` mode: frame→map homographies
  - `all` mode: all pairwise frame homographies + frame→map

### Part 2

#### Camera Calibration

- `calib_parameters/output/calib_L.mat` - left camera calibration
- `calib_parameters/output/calib_R.mat` - right camera calibration
- Each contains: cameraMatrix, distCoeff, rvecs, tvecs, objpoints, imgpoints

#### Neural Network Depth (compute_transform_NN.py)

- `output/NN/Disparity_Map.jpeg` - colorized disparity visualization
- `output/NN/Disparity_Map2.jpeg` - raw disparity with colorbar
- `output/NN/Depth_Map.jpeg` - normalized depth map
- `output/NN/Point_Cloud.pcd` - 3D point cloud (Open3D format)

#### Stereo Matching Depth (compute_transform_STEREO.py)

- `output/STEREO/Matched_SIFT.jpeg` - matched keypoints visualization
- `output/STEREO/Epipolar_Lines.jpeg` - epipolar geometry verification
- `output/STEREO/Rectified_Images.jpeg` - stereo-rectified image pair
- `output/STEREO/Disparity_Map.jpeg` - filtered disparity map (WLS filter)
- `output/STEREO/Depth_Map.jpeg` - computed depth map
- `output/STEREO/Point_Cloud.pcd` - 3D point cloud (Open3D format)

## Notes

- **Part 1 Note**: The script `compute_transform.py` is referenced as `compute_homography.py` in the original documentation - the actual filename is `compute_transform.py`.
- **Part 2 Note**: Camera calibration (`camera_calibration.py`) must be run before depth reconstruction scripts, as they depend on the calibration matrices.
- **Part 2 Note**: The neural network approach requires the ONNX model file. Download from the repository's `models/` directory if not present.
- **Part 2 Note**: `stereo_model.py` must be present alongside `compute_transform_NN.py` for the neural network approach.
- **Chessboard Parameters**: Hardcoded in `camera_calibration.py` - modify if using different calibration pattern.
- **Frame Size**: Both parts resize images to 600x800; adjust in code if using different resolution.
- **SIFT Parameters**: Default `nfeatures=0` (unlimited) in Part 1; ratio test threshold 0.5 in both parts.
