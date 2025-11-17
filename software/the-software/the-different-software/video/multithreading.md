---
description: How the video program uses multithreading for camera processing
---

# Multithreading

The Video program is designed to handle multiple camera feeds at the same time. To achieve this, it uses Dart's `Isolate` mechanism, which allows different parts of the application to run in parallel, each with its own memory space.

## Isolate-based Architecture

Our multithreading setup is built around a parent-child isolate pattern:

* Camera Manager (Parent Isolate):
  * This runs on the main application thread.
  * Its job is to create and oversee individual camera isolates.
  * It listens for video commands from the Dashboard and sends them to the correct child isolate.
  * It receives processed data (like frames, logs, and detected objects) from the child isolates and then forwards this information to the Dashboard or other rover programs (such as Autonomy or Computer Vision).
  * It handles all network communication, as only this parent isolate has direct access to network sockets.

* Camera Isolate (Child Isolates):
  * Each camera isolate runs on its own separate thread, complete with its own event loop and memory.
  * It manages a single camera device, whether it's an OpenCV-compatible USB camera or a RealSense camera.
  * Its responsibilities include initializing the camera, capturing frames, processing images (for example, detecting ArUco markers), and sending data back to the parent isolate.
  * This isolation is key: if one camera is slow or gets stuck, it won't affect the performance of other cameras or the main application.

## Communication Between Isolates

The parent camera manager and its child camera isolates talk to each other using message passing. This is a secure way to exchange data between isolates without sharing memory, which helps us avoid common issues like race conditions.

* Parent to Child: The camera manager sends video commands to child isolates to control camera settings (like starting/stopping streams, changing resolution, or taking snapshots).
* Child to Parent: Child isolates send payload messages back to the camera manager. These messages can contain:
  * Frame Payload: This includes a pointer to a native image buffer (like a JPG frame) and camera details. By sending just the pointer, we avoid copying large image data between isolates, which saves a lot of memory.
  * Log Payload: These are log messages that the parent's logger will process and forward.
  * Object Detection Payload: This contains details about objects detected (such as ArUco tags) and which camera spotted them.
  * Depth Frame Payload: This is a pointer to a native depth frame buffer, specifically for RealSense depth data.

## Benefits of Multithreading

* Concurrency: We can process multiple cameras simultaneously.
* Responsiveness: No single camera operation can freeze the entire application.
* Isolation: If a child isolate encounters an error or crashes, it's contained and doesn't bring down the whole video program.
* Efficiency: We reduce memory copying by passing pointers to native buffers between isolates, especially for large image frames.

This multithreaded architecture helps the Video program reliably meet the demands of managing multiple cameras.
