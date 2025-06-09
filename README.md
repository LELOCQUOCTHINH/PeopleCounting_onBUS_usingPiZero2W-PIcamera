# People Counting on Bus Project

## Overview
This project implements a people counting system for buses using a Raspberry Pi Zero 2W as the edge computing device, a 5MP Pi Camera for video capture, and three different algorithms for person detection: YOLO-tiny V4, Mobile SSD, and OpenCV self-build (background subtraction with MOG2). The system tracks passengers entering and exiting the bus, calculates the number of people inside, and monitors system resources (CPU, memory, temperature, and FPS). Processed data is sent to a CoreIoT dashboard via MQTT for real-time visualization. The project optimizes performance for the resource-constrained Raspberry Pi Zero 2W using low-resolution processing (320x240) and threaded frame reading.

## Repository Contents
- **yolo_counter.py**: Main script for processing video using YOLO-tiny V4 for person detection, tracking, counting, and sending telemetry to CoreIoT.
- **ssd_counter.py**: Main script for processing video using Mobile SSD for person detection, tracking, counting, and sending telemetry to CoreIoT.
- **opencv_counter.py**: Main script for processing video using OpenCV’s MOG2 background subtraction for person detection, tracking, counting, and sending telemetry to CoreIoT.
- **postTelemetry_mqtt_tb.py**: Utility script for MQTT communication with CoreIoT to send telemetry data.
- **Person.py**: Defines `MyPerson` and `MultiPerson` classes for tracking individuals and groups based on centroids and movement direction.
- **tracker.py**: Implements a `Tracker` class for assigning and updating object IDs based on centroid distances.
- **test_videos/**: Directory containing test videos (`test1.mp4`, `test2.mp4`, `test3.mp4`) for evaluating algorithm performance.

## Features
- **Person Detection**:
  - **YOLO-tiny V4**: Lightweight deep learning model optimized for edge devices, offering high accuracy (~80%+) but requiring frame skipping (e.g., frame skip = 5) on Raspberry Pi Zero 2W.
  - **Mobile SSD**: Deep learning model with very high accuracy but low performance (FPS ~3) due to high resource demands, often causing processing delays on Raspberry Pi Zero 2W.
  - **OpenCV Self-build**: Uses MOG2 background subtraction for lightweight detection, achieving very high performance (FPS ~7-10 with frame skip = 0) but lower accuracy (~60%) due to sensitivity to lighting and object confusion.

![Screenshot 2025-06-06 173623](https://github.com/user-attachments/assets/ed70e497-547a-49c8-b922-584d0f20015c)

- **Tracking**: Centroid-based tracking with `MyPerson` class assigns unique IDs and monitors movement across two virtual lines (upper for “Out,” lower for “In”).
- **Counting Logic**: Counts passengers crossing lines to determine entries and exits, calculating the number of people inside the bus.
- **Telemetry**: Sends real-time data (entry/exit counts, people inside, CPU usage, memory usage, temperature, FPS) to a CoreIoT dashboard.
- **Resource Monitoring**: Tracks CPU, memory, and temperature on the Raspberry Pi Zero 2W to ensure system stability.
- **Threaded Frame Reading**: Custom `PiCameraReader` and `VideoReader` classes use threading for efficient frame capture.
- **Frame Optimization**: Processes frames at 320x240 resolution to reduce computational load.

## Hardware Requirements
- **Raspberry Pi Zero 2W**: Edge computing device with quad-core Cortex-A53 CPU (1 GHz), 512MB RAM, WiFi, and Bluetooth.
- **Pi Camera Module (5MP)**: Captures video at 1080p@30fps or 720p@60fps, with a 53.5° field of view and auto white balance.
- **Internet Connection**: Required for MQTT communication with CoreIoT.

## Software Requirements
- **Python 3.7+**
- **OpenCV**:
  ```bash
  pip install opencv-python
  ```
- **Picamera2**:
  ```bash
  pip install picamera2
  ```
- **Paho MQTT**:
  ```bash
  pip install paho-mqtt
  ```
- **Psutil**:
  ```bash
  pip install psutil
  ```
- **Imutils**:
  ```bash
  pip install imutils
  ```
- **YOLO-tiny V4 Dependencies**:
  - Pre-trained weights and configuration files (download from the official YOLO repository).
  - Install `numpy` and `tensorflow` (lite version for edge devices):
    ```bash
    pip install numpy tensorflow
    ```
- **Mobile SSD Dependencies**:
  - Pre-trained model and configuration files (download from TensorFlow Model Zoo).
  - Install `tensorflow` (lite version):
    ```bash
    pip install tensorflow
    ```

## Setup Instructions
1. **Install Dependencies**:
   Install all required Python packages using the commands above.
2. **Configure CoreIoT**:
   - Create an account at `coreiot.io` and set up a device.
   - Obtain the **device access token**, server IP, and MQTT port (default: 1883).
3. **Connect Pi Camera**:
   - Connect the 5MP Pi Camera to the Raspberry Pi Zero 2W via the CSI port.
   - Enable the camera interface in `raspi-config`.
4. **Download Model Files**:
   - For YOLO-tiny V4, download weights and configuration files from the official repository.
   - For Mobile SSD, download the model from TensorFlow Model Zoo.
   - Place model files in the project directory and update paths in `yolo_counter.py` and `ssd_counter.py`.
5. **Run the Script**:
   - For YOLO-tiny V4 with Pi Camera:
     ```bash
     python3 yolo_counter.py --server-IP <COREIOT_IP> --Port <MQTT_PORT> --token <DEVICE_TOKEN>
     ```
   - For Mobile SSD with Pi Camera:
     ```bash
     python3 ssd_counter.py --server-IP <COREIOT_IP> --Port <MQTT_PORT> --token <DEVICE_TOKEN>
     ```
   - For OpenCV self-build with Pi Camera:
     ```bash
     python3 opencv_counter.py --server-IP <COREIOT_IP> --Port <MQTT_PORT> --token <DEVICE_TOKEN>
     ```
   - For testing with a video file (e.g., `test1.mp4`):
     ```bash
     python3 <script_name>.py --input test_videos/test1.mp4 --server-IP <COREIOT_IP> --Port <MQTT_PORT> --token <DEVICE_TOKEN>
     ```
6. **View Dashboard**:
   - Access the CoreIoT dashboard to monitor:
     - Number of people entering (`entered_people`)
     - Number of people exiting (`exited_people`)
     - Current people inside (`people_inside`)
     - System metrics (CPU usage, memory usage, temperature, FPS)
     - Performance charts over time (last 5 hours).

## Algorithm Evaluation
The project evaluates three algorithms using three test videos with varying conditions:
| Test Case | FPS | Duration | Total People | Total Out | Total In | Characteristics |
|-----------|-----|----------|--------------|-----------|----------|----------------|
| test1.mp4 | 30  | 42s      | 10           | 3         | 7        | Up to 4 people at once |
| test2.mp4 | 30  | 69s      | 16           | 14        | 2        | Few people with objects |
| test3.mp4 | 15  | 21s      | 14           | 14        | 0        | Many people, crowded |

### Model Comparison
| Model           | Accuracy | Performance | Notes |
|-----------------|----------|-------------|-------|
| YOLO-tiny V4    | High (~80%+) | High (7-10 FPS at frame skip=5) | Balanced accuracy and performance, suitable for real-time deployment. |
| Mobile SSD      | Very High | Low (3 FPS at frame skip=5) | High accuracy but resource-intensive, causing delays on Pi Zero 2W. |
| OpenCV Self-build | Low (~60%) | Very High (7-10 FPS at frame skip=0) | Lightweight but less accurate due to lighting sensitivity and object confusion. |

### Conclusion
YOLO-tiny V4 is selected as the most suitable algorithm for real-time deployment due to its balance of high accuracy (~80%+) and acceptable performance (7-10 FPS at frame skip=5), outperforming Mobile SSD (high accuracy but low FPS) and OpenCV self-build (high performance but low accuracy).

## Dashboard
The CoreIoT dashboard displays:
- **People Metrics**:
  - Number of people entering, exiting, and inside the bus.
  - Time-series chart of people counts.
- **System Metrics**:
  - Temperature (°C), CPU usage (%), memory usage (%), and FPS.
  - Time-series chart of system performance (last 5 hours) with colored lines for temperature (blue), CPU (green), memory (purple), and FPS (red).

## Usage Notes
- **Camera Placement**: Position the 5MP Pi Camera above the bus entrance/exit to capture clear passenger movement.
- **Algorithm Selection**: Use YOLO-tiny V4 for production due to its balance of accuracy and performance. OpenCV self-build is suitable for low-resource environments with controlled lighting.
- **Frame Skipping**: Adjust frame skip in `yolo_counter.py` and `ssd_counter.py` (default=5) to balance performance and accuracy.
- **Performance Tuning**: Adjust `areaTH` in `opencv_counter.py` for MOG2 sensitivity. Lower resolution or frame rate for better performance on Pi Zero 2W.
- **Exit**: Press `Esc` or `Ctrl+C` to stop scripts, displaying average resource usage and FPS.

## Limitations
- **Single Camera**: Assumes one entry/exit point. Multiple points require additional cameras.
- **Lighting Sensitivity**: OpenCV self-build is sensitive to lighting changes, affecting accuracy.
- **Resource Constraints**: Pi Zero 2W limits deep learning model performance, requiring frame skipping for YOLO-tiny V4 and Mobile SSD.
- **No Data Logging**: Telemetry is sent to CoreIoT but not logged locally. Add CSV logging for offline analysis.

## Future Improvements
- Implement local CSV logging for entry/exit events with timestamps.
- Enhance OpenCV self-build with adaptive thresholding or shadow removal.
- Support multiple cameras for multiple entry/exit points.
- Optimize deep learning models further for Pi Zero 2W (e.g., quantization).
- Integrate `MultiPerson` class for group tracking.

## License
This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## Contact
For questions or contributions, please contact [thinhle.hardware@gmail.com](mailto:thinhle.hardware@gmail.com) or
[My Linkedin](https://www.linkedin.com/in/lelocquocthinh/) or open an issue on the repository.

*LeLocQuocThinh/ 06-2025 - © <a href="https://github.com/LELOCQUOCTHINH" target="_blank">LLQT</a>.*
