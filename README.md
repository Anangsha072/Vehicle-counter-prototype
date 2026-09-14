# Vehicle Counting Using YOLO11 and ByteTrack

A computer vision project for detecting, tracking, and counting vehicles in a highway video using **YOLO11** and **ByteTrack**.

The system counts vehicles based on their movement across two virtual counting lines and provides separate counts for vehicles moving **towards the camera** and **away from the camera**.

## Overview

The project processes the video frame by frame and:

- Detects vehicles using YOLO11.
- Tracks vehicles across frames using ByteTrack.
- Calculates each vehicle's center point internally.
- Uses two virtual lines to determine movement direction.
- Counts a vehicle only after it crosses both lines.
- Counts the vehicle when the **second line** is crossed.
- Uses confidence-weighted class voting to determine the final vehicle class.
- Maintains separate counts for vehicles moving towards and away from the camera.
- Displays a live vehicle-count panel on the processed video.
- Generates a processed output video.

Bounding boxes, center points, and tracking ID labels are intentionally hidden in the final output, while the detection, tracking, and counting logic continues to work internally.

## Vehicle Classes

| Class ID | Vehicle |
|---:|---|
| 1 | Bicycle |
| 2 | Car |
| 3 | Motorcycle |
| 5 | Bus |
| 7 | Truck |

## Technologies Used

- Python
- YOLO11
- Ultralytics
- ByteTrack
- OpenCV
- Google Colab
- IPython

## Counting Logic

Two horizontal virtual lines are used:

- **Pink line** – first line for vehicles moving towards the camera.
- **Yellow line** – second line for vehicles moving towards the camera.

### Towards Camera

A vehicle moving downwards follows:

```text
PINK → YELLOW
```

The vehicle is counted as **Towards Camera** when its center point first crosses the Pink line and later crosses the Yellow line.

### Away From Camera

A vehicle moving upwards follows:

```text
YELLOW → PINK
```

The vehicle is counted as **Away From Camera** when its center point first crosses the Yellow line and later crosses the Pink line.

The vehicle is counted only when the **second line is crossed**. This helps distinguish the direction of movement and reduces incorrect counts from vehicles that only approach one line.

## Center-Point Based Counting

For every tracked vehicle, the center of its bounding box is calculated internally:

```python
cx = (x1 + x2) // 2
cy = (y1 + y2) // 2
```

The previous and current Y-coordinates are compared to detect line crossings.

The center point is used for the counting logic but is not displayed in the final processed video.

## Class Voting

The detected class of a vehicle can occasionally change between frames. To make the final class more stable, the project accumulates confidence scores for each class associated with a tracking ID.

At the time the vehicle is counted, the class with the highest accumulated confidence score is selected as the final class.

This helps reduce the effect of occasional frame-level classification changes.

## ByteTrack Configuration

The project creates a custom ByteTrack configuration:

```yaml
tracker_type: bytetrack
track_high_thresh: 0.15
track_low_thresh: 0.05
new_track_thresh: 0.15
track_buffer: 60
match_thresh: 0.80
fuse_score: True
```

YOLO tracking is run with:

```python
model.track(
    frame,
    persist=True,
    tracker=TRACKER_CONFIG,
    classes=VEHICLE_CLASS_IDS,
    conf=0.35,
    iou=0.50,
    imgsz=1280,
    verbose=False
)
```

## Counting State Machine

Each tracked vehicle has an internal state used to determine the order of line crossings.

Main states include:

- `POTENTIAL_DOWN`
- `POTENTIAL_UP`
- `BETWEEN`
- `PINK_FIRST`
- `YELLOW_FIRST`
- `COUNTED_DOWN`
- `COUNTED_UP`

The two main transitions are:

```text
POTENTIAL_DOWN
      ↓
PINK_FIRST
      ↓
YELLOW crossed
      ↓
COUNTED_DOWN
```

and:

```text
POTENTIAL_UP
      ↓
YELLOW_FIRST
      ↓
PINK crossed
      ↓
COUNTED_UP
```

Separate tracking-ID sets are used so that a vehicle is not counted repeatedly in the same direction.

## Counting Lines

The line positions are calculated relative to the video height:

```python
PINK_LINE_Y = int(height * 0.50)
YELLOW_LINE_Y = int(height * 0.65)
```

Because the positions are based on the video height, they automatically scale with different video resolutions.

These percentages can be adjusted if the camera angle or road layout changes.

## Processing Workflow

```text
Input Highway Video
        ↓
Read Frame
        ↓
YOLO11 Vehicle Detection
        ↓
ByteTrack Tracking
        ↓
Get Track ID + Vehicle Class
        ↓
Calculate Center Point
        ↓
Update Class Confidence Scores
        ↓
Check Previous/Current Position
        ↓
Detect Line Crossing
        ↓
Update Vehicle State
        ↓
Second Line Crossed?
      ↙       ↘
    YES        NO
     ↓          ↓
   Count     Continue Tracking
     ↓
Update Direction + Vehicle Class
        ↓
Draw Lines + Counter Panel
        ↓
Save Output Frame
        ↓
Final Output Video
```

## Counter Panel

The output video displays separate counts for:

- Car
- Motorcycle
- Bus
- Truck
- Bicycle

It also displays the total number of vehicles moving in each direction.

## Input and Output

### Input Video

The notebook expects the input video at:

```text
/content/video.mp4
```

### Output Video

The processed video is saved as:

```text
/content/vehicle_counting_output.mp4
```
## Demo / Output

Here is a short demonstration of the vehicle detection and directional
counting system:

![Vehicle Counting Demo](<img width="1795" height="798" alt="image" src="https://github.com/user-attachments/assets/3e7a3909-d74c-4e13-9602-7416a15a89e6" >)


The notebook also displays the completed video and provides a Google Colab download option.

## Installation

Install the required packages with:

```bash
pip install ultralytics opencv-python
```

The project is designed to run in Google Colab.

## How to Run

1. Open the notebook in Google Colab.
2. Upload the highway/input video.
3. Make sure `yolo11l.pt` is available.
4. Run the notebook cells.
5. The custom ByteTrack configuration is created automatically.
6. The video is processed frame by frame.
7. The live counter is updated during processing.
8. Final counts are printed after processing.
9. The processed video is displayed and can be downloaded.

## Important Parameters

```python
MODEL_PATH = "yolo11l.pt"

PINK_LINE_Y = int(height * 0.50)
YELLOW_LINE_Y = int(height * 0.65)

conf = 0.35
iou = 0.50
imgsz = 1280
```

### Adjusting Counting Lines

If vehicles are being missed because they do not reach a counting line, adjust the values of:

```python
PINK_LINE_Y
YELLOW_LINE_Y
```

### Adjusting Detection Confidence

The current detection confidence is `0.35`. Lowering it may detect more vehicles but can increase false detections. Increasing it can reduce false detections but may miss some vehicles.

## Features

- [x] YOLO11 vehicle detection
- [x] ByteTrack multi-object tracking
- [x] Unique tracking IDs
- [x] Center-point based line crossing
- [x] Two-line directional counting
- [x] Towards-camera counting
- [x] Away-from-camera counting
- [x] Car counting
- [x] Motorcycle counting
- [x] Bus counting
- [x] Truck counting
- [x] Bicycle counting
- [x] Confidence-weighted class voting
- [x] Duplicate-count prevention
- [x] Live processing preview
- [x] On-video counter panel
- [x] Final count summary
- [x] Output video generation

## Duplicate Count Prevention

The project uses two sets of tracking IDs:

```python
counted_down_ids = set()
counted_up_ids = set()
```

Before a vehicle is added to a count, its tracking ID is checked. This prevents the same tracked vehicle from being counted multiple times in the same direction.

## Why Two Lines?

A single counting line can tell whether an object crossed a location, but it does not reliably describe the intended direction in this setup. Using two lines allows the project to identify the order of movement:

```text
PINK → YELLOW = Towards Camera
YELLOW → PINK = Away From Camera
```

The vehicle is counted only after the second line is crossed.

## Recommended GitHub Structure

```text
vehicle-counting/
│
├── vehicle_counting.ipynb
├── README.md
```
## Demo / Output

The following video shows the vehicle detection and directional counting result:

![Vehicle Counting Demo]


Large input/output videos and model files should generally not be committed directly to the repository.

## Limitations

Counting accuracy depends on:

- Video quality
- Camera angle
- Vehicle visibility
- Occlusion between vehicles
- YOLO detection accuracy
- ByteTrack maintaining consistent tracking IDs
- Correct placement of the counting lines

For videos recorded from a different camera position, the counting-line percentages may need to be adjusted.

## Future Improvements

Possible extensions include:

- Automatic counting-line selection
- Region-of-interest filtering
- Vehicle speed estimation
- Vehicle trajectory visualization
- Traffic density analysis
- Time-based traffic statistics
- CSV/Excel export
- Traffic analytics dashboard
- Better handling of heavily occluded vehicles
- Multiple counting zones

## Author

**Anangsha Das**

A computer vision project for vehicle detection, multi-object tracking, and directional traffic counting using YOLO11 and ByteTrack.
