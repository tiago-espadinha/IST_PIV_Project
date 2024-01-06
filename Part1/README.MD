# Part 1: Video Processing & Homography Estimation

## Team Members

- Diogo Rosa, ist93044
- Tiago Simões, ist96329
- Tomás Santos, ist100099
- Filipe Tendeiro, ist108325

## Project Description

This module processes video files to extract frames, detects SIFT keypoints and descriptors, enables manual keypoint selection for reference mapping, and computes homographies between frames using RANSAC-based robust estimation. It serves as the foundation for video stabilization, panorama creation, and frame-to-map alignment tasks.

## Requirements

- configparser
- math
- matplotlib
- numpy
- opencv-python (cv2)
- os
- random
- scipy
- sys

### Data Requirements

- Video file (e.g., `trymefirst.mp4`) - specify path in `conf_file.cfg`

## Installation

1. Install dependencies as specified above:

```bash
pip install numpy scipy matplotlib opencv-python
```

1. Prepare data files as required
2. Configure any necessary settings

## Usage

### Configuration

Edit `conf_file.cfg` to set the following parameters:

| Parameter              | Description                                | Example                               |
| ---------------------- | ------------------------------------------ | ------------------------------------- |
| `videos`               | Path to input video file                   | `trymefirst.mp4`                      |
| `frames_directory`     | Output directory for frames                | `processed_videos/trymefirst/frames/` |
| `image_map`            | Reference map image name                   | `map.jpg`                             |
| `undersampling_factor` | Frame sampling interval                    | `30` (every 30th frame)               |
| `keypoints_out`        | SIFT features output file                  | `features.mat`                        |
| `transforms_out`       | Homography output file                     | `homography.mat`                      |
| `pts_in_map`           | Map keypoints (set via get_keypoints.py)   | `Label 165 187 364 62...`             |
| `pts_in_frame`         | Frame keypoints (set via get_keypoints.py) | `30 257 118 438...`                   |
| `transforms`           | Mode: `homography map` or `homography all` | `homography map`                      |

### Running the Project

#### 1. Process Video (`process_video.py`)

Extracts frames at the specified undersampling rate and computes SIFT features for each.

```bash
python process_video.py [config_file.cfg]
```

- If no config provided, defaults to `conf_file.cfg`
- Saves frames to `frames_directory`
- Saves SIFT features to `keypoints_out` (MATLAB .mat format)
- Press 'q' to stop early

#### 2. Get Keypoints (`get_keypoints.py`)

**Only run if you need to update reference keypoints in the config file.**

```bash
python get_keypoints.py [config_file.cfg]
```

- Opens two windows: map image (map.jpg) and a reference frame (frame_0030.jpg by default)
- Click ≥4 corresponding points in each image (left-click)
- Press any key when done
- Updates `pts_in_map` and `pts_in_frame` in config file automatically

#### 3. Compute Transform (`compute_transform.py`)

Computes homographies using RANSAC for robust estimation.

```bash
python compute_transform.py [config_file.cfg]
```

**Modes** (set via `transforms` in config):

- `homography map` - computes frame-to-map homographies for all frames
- `homography all` - computes all pairwise frame homographies + frame-to-map

Output saved to `transforms_out` (MATLAB .mat format).

## Outputs

### Features (`features.mat`)

MATLAB struct array with one entry per processed frame:

- `keypoints`: Nx2 array of (x, y) coordinates
- `descriptors`: Nx128 SIFT descriptor vectors

### Homographies (`homography.mat`)

MATLAB struct containing transformation matrices:

- **Map mode**: Array of [frame_idx, 0, h11, h12, ..., h33] for frame→map
- **All mode**: Includes above plus [frame_i, frame_j, h11, ..., h33] for all pairs

## Notes

- The script is named `compute_transform.py` (not `compute_homography.py` as referenced in some documentation)
- SIFT feature detection uses unlimited keypoints (`nfeatures=0`)
- Ratio test threshold for FLANN matching: 0.5
- RANSAC parameters: P=0.99, p=0.5, 4 samples, tolerance=3.0 pixels
- Minimum 20 good matches required for direct homography; otherwise uses recursive midpoint chaining
