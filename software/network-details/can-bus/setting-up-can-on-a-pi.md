---
description: The Subsystems Computer's interface to the firmware
---

# Setting up CAN on a Pi

Linux has support for the CAN protocol when using an external chip, but it needs to be enabled and configured before it can be used. We use the `` MCP2518FD` `` chip on a custom-built Pi hat to achieve this, and this page describes how to set up the chip on a fresh Raspberry Pi.

## The CAN0 overlay

First, we need to tell the Pi how to interface with the chip. The Raspberry Pi works with a "device tree" that tells it what hardware to use and how to find it. We will add our own file to this tree and modify the boot config to expose this device to the operating system.

First, add the following to a new file, titled `mcp2518fd-can0.dts:`

```groovy
// TODO: Find this code
```

Compile this file to a `.dtbo` and copy it to the boot directory:

```bash
dtc -@ -I dts -O dtb mcp2518fd-can0.dts -o mcp2518fd-can0.dtbo
cp mcp2518fd-can0.dtbo /boot/overlays
```

Now add the following lines to the end of `/boot/config.txt`:

```toml
# MCP2518FD CAN chip support
# TODO: Verify this
[all]
dtparam=spi=on
dtoverlay=mcp2518fd-can0,oscillator=16000000,interrupt=25,spimaxfrequency=1000000
```

## The command line

Reboot the Pi. Now that the hardware has been configured, it's time to set up CAN as a network device in the operating system.

First, update your software packages:

```bash
sudo apt update
sudo apt upgrade
sudo reboot
```

Then, use the ip tool to register CAN:

```bash
sudo ip link add can0
sudo ip link set can0 up type can bitrate 500000
```

The bitrate must match the one registered in `config.txt` and the one used on the firmware. The `add` command needs to be run just once, but the `set` command needs to be run on every boot. This can also be done by the Subsystems program.

## Testing

The `can-utils` package offers tools to quickly test the CAN bus. Run `sudo apt install can-utils` to install the tools, then use `cansend` to send data and `candump` to receive data. One quick test is to open two terminals and run `candump can0` in one and `cansend can0#deadbeef` in the other. Note that at least one other CAN-capable device must be connected and listening for messages in order for the bus to work.

## More Resources

* [The MCP2518FD chip](https://www.microchip.com/en-us/product/mcp2518fd)
* [Raspberry's docs on Device Trees](https://www.raspberrypi.com/documentation/computers/configuration.html#changing-the-default-pin-configuration)
* [Using `dts` and overlays](https://blog.stabel.family/raspberry-pi-4-device-tree/)
* The [`can-utils`](https://github.com/linux-can/can-utils) tool
