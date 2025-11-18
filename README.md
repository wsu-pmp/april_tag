# AprilTag Detection Pipeline (ROS 2 Humble)

### Using ZED 2i as USB Camera (`/dev/video0`)

* **v4l2_camera** — streams ZED 2i as a raw USB camera
* **apriltag_ros** — detects AprilTags in incoming images
* **ros2 topic echo** — verifies detections

Workspace:

```
~/Documents/ros2_ws
```

---

## 0. Environment Setup

### 0.1 Deactivate Conda (required for ROS)

Run this in **every** ROS terminal:

```bash
conda deactivate
```

### 0.2 Source ROS + Workspace

```bash
source /opt/ros/humble/setup.bash
source ~/Documents/ros2_ws/install/setup.bash
```

---

## 1. Install & Build `apriltag_ros` (One-time Setup)

### 1.1 Clone the package

```bash
cd ~/Documents/ros2_ws/src
rm -rf apriltag_ros
git clone https://github.com/christianrauch/apriltag_ros.git
```

### 1.2 Install dependencies

```bash
cd ~/Documents/ros2_ws
rosdep install --from-paths src --ignore-src -r -y
```

### 1.3 Build only the AprilTag package

```bash
cd ~/Documents/ros2_ws
colcon build --symlink-install --packages-select apriltag_ros
```

After building, always re-source:

```bash
source /opt/ros/humble/setup.bash
source ~/Documents/ros2_ws/install/setup.bash
```

---

## 2. Create AprilTag Config File

This config:

* Detects **36h11** tags
* Disables pose estimation (no calibration needed)
* Uses safe detection parameters

Create file:

```bash
mkdir -p ~/Documents/ros2_ws/config/apriltag

cat << 'EOF' > ~/Documents/ros2_ws/config/apriltag/simple_tags.yaml
apriltag:
  ros__parameters:
    image_transport: "raw"
    family: "36h11"
    size: 0.12

    # disable pose estimation until camera is calibrated
    pose_estimation_method: ""

    max_hamming: 0
    detector:
      threads: 2
      decimate: 2.0
      blur: 0.0
      refine: true
      sharpening: 0.25
      debug: false
EOF
```

---

# 3. Runtime Instructions (3 Terminals)

## Terminal 1 — Start the Camera (ZED 2i USB Mode)

```bash
conda deactivate
source /opt/ros/humble/setup.bash
source ~/Documents/ros2_ws/install/setup.bash

ros2 run v4l2_camera v4l2_camera_node --ros-args \
  -p video_device:=/dev/video0 \
  -p fps:=15
```

Topics produced:

```
/image_raw
/camera_info
```

---

## Terminal 2 — Start AprilTag Detector

```bash
conda deactivate
source /opt/ros/humble/setup.bash
source ~/Documents/ros2_ws/install/setup.bash

ros2 run apriltag_ros apriltag_node --ros-args \
  -r image_rect:=/image_raw \
  -r camera_info:=/camera_info \
  --params-file /home/wsu/Documents/ros2_ws/config/apriltag/simple_tags.yaml
```

This subscribes to:

* `/image_raw`
* `/camera_info`

And publishes:

* `/detections`

---

## Terminal 3 — View AprilTag Detections

```bash
conda deactivate
source /opt/ros/humble/setup.bash
source ~/Documents/ros2_ws/install/setup.bash

ros2 topic echo /detections
```

Hold a printed **36h11** AprilTag → you should see:

* `id`
* `hamming`
* `decision_margin`

(poses are disabled intentionally)

### SAMPLE of detecting tag id 4 : 

<img width="300" height="800" alt="image" src="https://github.com/user-attachments/assets/b891ff3f-0da1-45fd-974f-c69c6904df9f" />
<img width="300" height="300" alt="image" src="https://github.com/user-attachments/assets/1b1e82d1-38a9-4e49-9a11-1594e8ab1e65" />

---

# Summary Diagram

```
ZED 2i (USB)
/dev/video0
      ↓
v4l2_camera_node
      ↓ publishes
  /image_raw + /camera_info
      ↓
apriltag_node
      ↓ publishes
    /detections
      ↓
ros2 topic echo /detections
```

---



