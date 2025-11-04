---
description: How the video program processes and streams Lidar data
---

# Lidar Streaming

The Video program also takes on the responsibility of receiving, processing, and distributing data from the Lidar sensor. This Lidar data gives both the Dashboard and the Autonomy program vital spatial awareness. Our Lidar manager handles every aspect of this data stream.

## Lidar Data Acquisition

The Lidar manager operates as an independent component within the Video program. Its main duties include:

* UDP Socket Listener: It sets up a UDP socket on a dedicated port (8004) to listen for incoming data packets sent by the external Lidar program.
* Data Reception: It constantly monitors this UDP socket for incoming datagram packets.
* Data Parsing: When a packet arrives, it parses the raw `Float64List` data. The first 8 bytes of the packet tell us whether the data that follows represents Cartesian or Polar coordinates.
* Coordinate Conversion: Based on this flag, the raw `Float64List` is converted into a list of either Cartesian or Polar protobuf messages.
  * Cartesian Points: These are processed as (x, y) pairs.
  * Polar Points: These are processed as (angle, distance) pairs. We make a specific adjustment to the angle range (shifting it from `[0, 270]` to `[-135, +135]`) to match the Lidar's physical orientation.
* Message Distribution: The processed Lidar data, neatly packaged into a point cloud protobuf message, is then sent to:
  * The Dashboard
  * The Autonomy program

## Data Format

We expect the Lidar data to arrive as a `Float64List`. The very first element in this list acts as a flag:

* `0x01`: This indicates that the subsequent data points are in Cartesian coordinates.
* Any other value: This tells us that the subsequent data points are in Polar coordinates.

Each individual data point (whether Cartesian or Polar) is then wrapped into its respective protobuf message and included in a larger point cloud message.

## Network Configuration

The Lidar manager listens for incoming UDP packets on port 8004. This means the external Lidar program must be set up to send its data to the Video program's IP address on this exact port. Once processed, these point cloud messages are then forwarded to the Autonomy program, which should be listening on `192.168.1.30:8003`.

## Troubleshooting

* No Lidar Data on Dashboard/Autonomy: First, confirm that your Lidar sensor is working and that its associated program is running and actively sending data. Then, check that the Lidar program is sending data to the Video program's correct IP address and port (8004).
* Incorrect Lidar Readings: Make sure the data format being sent by the Lidar program exactly matches what we expect (the first `Float64` as a flag, followed by coordinate data). If your polar readings seem off, double-check the angle conversion logic.
* Network Issues: Verify the UDP connectivity between the Lidar program, the Video program, and the Autonomy program. Also, ensure that no firewalls are blocking UDP traffic on the necessary ports.
