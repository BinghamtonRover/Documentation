---
description: How the video program streams from the Intel RealSense camera
---

# RealSense Streaming

The Video program seamlessly integrates with the Intel RealSense depth camera, giving us access to RGB, depth, and colorized depth streams. This capability is vital for applications like Autonomy that rely on detailed spatial information. We achieve this integration through FFI (Foreign Function Interface) bindings to the native RealSense SDK.

## RealSense Integration

A dedicated child isolate takes on the responsibility of managing the RealSense camera. It handles all the RealSense-specific logic:

* FFI Bindings: We use generated FFI bindings to make calls to the native C/C++ functions provided by the RealSense SDK.
* Initialization: Our initialization method creates a native RealSense object and gets the device ready. It also kicks off the RealSense stream, gathering the camera's depth and RGB resolutions, along with its depth scale.
* Frame Acquisition: A specific method retrieves pointers to the raw RGB, depth, and colorized depth data buffers.
* Frame Processing and Transmission: This component processes and sends three distinct types of frames:
  * Colorized Depth Frame: We convert the raw colorized depth data into an OpenCV `Mat`, encode it as a JPEG, and then send it off to the Dashboard.
  * RGB Frame: The raw RGB data is transformed into an OpenCV `Mat`, where we perform ArUco marker detection. This is then encoded as a JPEG and sent to the Dashboard. Any detected ArUco tags are also forwarded to the Autonomy program.
  * Depth Frame: The raw depth data goes directly to the Autonomy program as a depth frame payload.

## Autonomy Considerations

Because the RealSense camera is so important for the Autonomy program, we've put some restrictions on changing certain camera settings (like resolution and FPS). This prevents any changes that might disrupt Autonomy's operations. If someone tries to alter these critical settings, our system will log an error.

## Frame Data Handling

To keep things running smoothly and avoid unnecessary memory usage, the RealSense isolate works directly with pointers to native memory buffers. When frames come in from the RealSense SDK, they stay in native memory. Only their memory addresses are passed between the FFI layer and Dart. When we send frames to the parent camera manager isolate, we use a frame payload or depth frame payload, which simply contains this native memory address. The parent isolate then reads the data from this address and takes care of releasing the native memory once it's done.

## Configuration and Defaults

You'll find the default settings for the RealSense camera in `constants.dart`. These settings are carefully chosen to strike a balance between the needs of depth sensing for Autonomy and RGB streaming for the Dashboard. Just like with our other cameras, you can override RealSense settings using configuration files located in the `camera_details/` directory.

## Troubleshooting

* RealSense Not Detected/Initialized: First, check that your RealSense camera is properly connected and powered. Then, confirm that the `librealsense` SDK and its necessary components are correctly installed on your operating system. Also, take a look at the FFI configuration and the generated FFI bindings for any potential issues.
* No Depth Data/Incorrect Readings: Verify that the RealSense stream is starting up successfully. Check the scale value to ensure depth measurements are being interpreted correctly. And, of course, make sure the camera lens is clean and nothing is blocking it.
* Autonomy Interference: If you find that you can't change certain camera settings, it's likely due to our autonomy interference check. In this case, it's best to chat with the Autonomy team to understand their specific requirements for the RealSense camera.
* Frame Dropping/Lag: Similar to our OpenCV cameras, if you're experiencing frame drops or lag, try reducing the JPEG quality for RGB/colorized streams if bandwidth is a concern. Also, ensure your processing unit has enough resources to handle the load.
