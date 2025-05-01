---
description: Intregating SICK's TiM551-2050001 LiDAR Sensor
---

# Lidar Streaming

## Introduction
We integrated a [SICK TiM551-2050001 LiDAR](https://www.sick.com/us/en/catalog/products/lidar-and-radar-sensors/lidar-sensors/tim/tim551-2050001/p/p343045?tab=detail) to help with obstacle detection during the [autonomous mission](../../../../team-details/the-missions/autonomous-navigation.md "mention").
The LiDAR we are using provides a 270 field of view with a range of 0.05 to 10 meters.
The LiDAR program receives messages from the LiDAR and converts it to a JPG image which gets sent to the Dashboard, and an array of bytes that is sent to the autonomy program to be processed. 

## Details
The LiDAR repository makes use of dart's FFI compbatiblity by compiling C++ files then calling the C++ functions in dart. 
The C++ code makes use of the [Sick Scan API](https://github.com/SICKAG/sick_scan_xd) which is maintained by the manufactor of the LiDAR. 
The Sick Scan API provides functionality to communicate with the LiDAR and allows us to provide handler function(s) which get called automatically when the LiDAR receives new data.
Our program makes use of a single handler function which when called creates an image and populates an array with datapoints about what the LiDAR sees. 

The C++ code handles all communication directly with LiDAR device and data received including
* Initializing Sick Scan API's handle from a launch file
* Disposing of the handle
* Creating and maintaining an image
* Maintaining an array of 270 bytes that contain data for every degree the lidar sees

The dart code is the code that gets run and calls the C++ functions through FFI.  

## Where the data goes
The dart programs regularly gets the latest image from the C++ program and sends it over to the video program which then sends it to the Dashboard.
 
---- SENDING THE BYTE ARRAY TO AUTONOMY NOT YET DETERMINED ---- 


## Compiling the DLLs and FFI
Inside of the repository there is a batch file which will generate all the files needed to run the program.
```batch
~./build.bat
```
The build file creates a large `.dll` file which bundles the Sick Scan API and our C++ which is used when looking up function definition for dart's FFI.
The build file also compiles the FFI bindings and puts them in the correct place.

## Running the program
To run the program
```batch
dart run --enable-experiment=native-assets
```
**Note** The flag `enable-experiment=native-assets` might be redudant in future dart releases. 

## More Resources

* [TiM551-2050001 Product Data Sheet](https://cdn.sick.com/media/pdf/5/45/045/dataSheet_TIM551-2050001_1060445_en.pdf)
* [Sick Scan API](https://github.com/SICKAG/sick_scan_xd)
* [Sick Telegram Protocol](https://cdn.sick.com/media/docs/7/27/927/technical_information_telegram_listing_ranging_sensors_lms1xx_lms5xx_tim2xx_tim5xx_tim7xx_lms1000_mrs1000_mrs6000_nav310_ld_oem15xx_ld_lrs36xx_lms4000_lrs4000_en_im0045927.pdf)

