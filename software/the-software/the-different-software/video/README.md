---
description: How the rover's video program processes and streams camera and Lidar data
---

# Video Program

## Introduction

The Video program is at the heart of our rover's visual and spatial awareness. It's responsible for capturing, processing, and streaming video feeds from various cameras, alongside handling Lidar data. This central hub provides essential information to the Dashboard for operator control and feeds the Autonomy program with data crucial for navigation and object detection.

Built with Dart, the program cleverly uses isolates to process multiple camera streams simultaneously, ensuring smooth and efficient operation. It seamlessly integrates with OpenCV for standard USB cameras and taps into the native Intel RealSense SDK for advanced depth camera capabilities through FFI (Foreign Function Interface).

## Architecture

The Video program's design revolves around a `Collection` service, which acts as the main coordinator for both camera and Lidar management. Our camera system employs a parent-child isolate model: a main isolate sets up and oversees dedicated child isolates, with each child handling a specific camera. This setup allows every camera to work independently and in parallel, preventing bottlenecks and maximizing performance.

Here's a breakdown of how data flows within the Video program:

1. Camera Acquisition:
  * OpenCV Cameras: We manage standard USB cameras using specialized components that interface directly with OpenCV.
  * RealSense Camera: An isolate dedicated to the Intel RealSense depth camera communicates with the native RealSense SDK via FFI bindings. This provides us with RGB, depth, and colorized depth frames.
2.  Lidar Data: A dedicated Lidar manager listens for incoming UDP packets containing Lidar data. It processes this raw data and then forwards it as organized point cloud messages.
3.  Inter-Isolate Communication: Child camera isolates send their processed frames, detected objects, and log messages back to the parent camera manager using special payload messages. This smart approach avoids copying large image buffers by sending only pointers to native memory, which is a huge win for efficiency.
4. Network Communication:
  * Dashboard: Our main service talks to the Dashboard, sending video data (including frames, detected objects, and camera details) and receiving commands to control camera functions like resolution changes or taking snapshots.
  * Autonomy Program: Depth frames from the RealSense camera and processed Lidar data go directly to the Autonomy program via dedicated UDP sockets. We also forward detected ArUco tags to Autonomy.
  * Computer Vision Program: If configured, frames from certain cameras can be sent to a separate Computer Vision program for more in-depth analysis.

## Key Features

* Multi-camera Support: We handle multiple USB cameras and a RealSense depth camera all at once.
* Isolate-based Concurrency: Dart isolates enable parallel processing of camera feeds, boosting overall responsiveness.
* RealSense Integration: We provide access to RGB, depth, and colorized depth streams from the Intel RealSense camera.
* Lidar Data Processing: The program receives and processes Lidar point cloud data, transforming it into a consistent, standardized format.
* ArUco Marker Detection: We detect ArUco markers in camera feeds, calculate their 2D (yaw, pitch) and 3D (pose) positions, and can even overlay visual indicators on the frames.
* Dynamic Camera Control: The Dashboard can dynamically adjust camera settings like resolution, quality, FPS, zoom, and focus.
* Screenshot Capability: We can capture high-quality still images from cameras and save them to a shared network folder.
* Configurable Logging: Our integrated logger provides comprehensive logging, with messages conveniently forwarded to the Dashboard.
* Shared Configuration: Camera settings and calibration data are loaded from and saved to JSON files in a shared directory, ensuring consistency.

## Camera Management

Each camera operates under the care of its own isolate. This abstract structure provides a unified way to handle camera operations, while specific implementations manage the unique aspects of different camera types.

Camera settings, such as resolution, FPS, and image quality, can be fine-tuned through commands sent from the Dashboard. The program also supports saving and loading camera configurations from JSON files, so your settings persist across restarts.

## Object Detection and Targeting

The Video program includes an ArUco detector to pinpoint ArUco markers. This capability is vital for tasks that demand precise object localization and navigation. When markers are found, the program calculates:

* 2D Position: The yaw and pitch relative to the camera's center.
* 3D Pose: The translation and rotation vectors (camera-to-target) using the `solvePnP` algorithm, which relies on accurate camera calibration data.

Camera calibration data, stored in JSON files, provides the intrinsic parameters (like focal length and field of view) needed for these 3D calculations.

## Lidar Streaming

Our Lidar manager takes charge of receiving and interpreting data from the Lidar sensor. It listens on a dedicated UDP port (8004) for incoming packets. The raw Lidar data, typically a `Float64List` representing distances and angles, is then converted into structured point cloud protobuf messages. These messages can contain either Cartesian or Polar coordinates, depending on the Lidar sensor's output. Once processed, the Lidar data is streamed to both the Dashboard and the Autonomy program.

## Configuration

The Video program relies on several configuration files and constants:

* `camera_details/*.json`: These JSON files store individual camera settings (resolution, FPS, quality, etc.). They're loaded at startup and updated when settings change.
* `calibrations/<camera_name>/*.json`: These JSON files contain camera calibration coefficients (intrinsics and distortion coefficients) essential for accurate 3D pose estimation in ArUco detection.
* `constants.dart`: This file defines mappings for camera names to device paths (Linux) or indexes (Windows), sets default camera settings, and configures the default ArUco detector.

## Troubleshooting

* Camera Not Detected: Double-check camera connections and verify camera name mappings in `constants.dart` to ensure the correct device is being used.
* RealSense Initialization Failure: Make sure the RealSense SDK is properly installed and its FFI bindings are correctly configured for your operating system.
* Frame Dropping/Lag: Try adjusting camera `fps` and `quality` settings. High resolutions or quality can sometimes strain network bandwidth or processing capabilities.
* ArUco Detection Issues: Ensure markers are clearly visible within the camera's field of view, well-lit, and that the marker size in the default ArUco configuration is accurate. If 3D pose estimation seems off, verify your camera calibration data.
* Lidar Data Not Received: Confirm that the Lidar sensor is operational and its program is actively sending data. Check that the Lidar program is sending data to the Video program's correct IP address and port (8004).
