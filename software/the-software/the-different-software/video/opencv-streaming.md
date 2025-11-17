---
description: How the video program streams from OpenCV-compatible cameras
---

# OpenCV Streaming

The Video program uses OpenCV, powered by the `dartcv4` library, to connect with and stream from standard USB cameras. This section explains how we manage these cameras and process their video feeds.

## Camera Management

Each OpenCV-compatible camera connected to the rover is handled by its own specialized child isolate. This isolate takes care of:

* Initialization: It opens the camera device and sets up its initial configuration.
* Settings Configuration: It applies camera parameters like resolution, frames per second (FPS), zoom, focus, pan, tilt, roll, and autofocus. The Dashboard can dynamically update these settings.
* Frame Acquisition: It continuously reads frames from the camera.
* Image Processing: It performs tasks such as ArUco marker detection on the captured frames.
* Frame Encoding and Transmission: It encodes the processed frames into JPEG format and sends them to the parent camera manager isolate, which then forwards them to the Dashboard.

## Dynamic Camera Settings

Our OpenCV camera isolates allow for on-the-fly adjustments to various camera properties. When the Dashboard sends a command with updated camera details, the isolate applies these changes to the camera. This includes:

* Resolution
* FPS
* Zoom
* Focus
* Pan/Tilt/Roll
* Autofocus

If you change the resolution or FPS, the camera stream will restart to apply the new settings. Other settings are applied instantly.

## Frame Processing Workflow

1. Read Frame: A method within the camera isolate captures a new frame as an OpenCV `Mat` object.
2. ArUco Detection: An ArUco detector then processes this `Mat` to find any ArUco markers. Any detected markers and their calculated positions are sent to the parent isolate.
3. Resolution Adjustment: If the camera's actual capture resolution doesn't match the desired stream resolution (perhaps to save bandwidth), the frame is resized.
4. JPEG Encoding: The `Mat` object is converted into a JPEG using a specified quality setting. If the resulting JPEG is too large for the network, its quality is automatically reduced until it fits or reaches a minimum threshold.
5. Transmission: The JPEG frame (or a pointer to its native memory) is then sent to the parent camera manager.

## Configuration and Constants

We define camera device mappings (like `/dev/video0` on Linux or a numerical index on Windows) in `constants.dart`. This file also holds default camera settings for OpenCV cameras, which are used if no specific configuration is found or loaded from JSON files in the `camera_details/` directory.

## Troubleshooting

* Camera Not Opening: Double-check your camera connections and verify the camera name mappings in `constants.dart` to ensure the correct device is being used. To make sure the camera is connected, see if the path in `constants.dart` exists on the operating system, otherwise, it may not be connected or detected by UDev.
* Poor Performance/Lag: Try lowering the camera's resolution or FPS. You can also reduce the JPEG quality setting. Make sure the rover's processing unit has enough resources.
* Incorrect Camera Settings: Confirm that commands are being sent correctly from the Dashboard and that the camera settings are being applied as expected. Review the logs for any errors during setting updates.
