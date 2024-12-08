---
description: Reading non-human-readable Protobuf data
---

# Debugging Protobuf

## Introduction

Protobuf, as opposed to JSON and other formats, is non-human-readable. This means, as efficient as they are, buffers cannot be read just by looking at a log file or print statement (note that printing the _generated_ class/struct works fine, but not the serialized buffer). To solve this, we use a tool called [ProtobufPal](https://www.protobufpal.com/). Think of it as a pre-written script that can encode or decode your messages.

## Getting your buffer

As mentioned, if you're able to print or log your data in the code, this is always preferable as you can access individual fields on the class/struct. This form of debugging is when sending data _over the wire_, ie, through the network. If, for some reason, the data becomes corrupted along the way or you suspect it is otherwise not being parsed correctly, you can inspect the raw buffer instead.

These issues may be caused by:&#x20;

* a broken network setup, such as a loose CAN wire
* data changing before it is actually sent
* out-of-sync generated code on each endpoint
* default values not being sent

Note that the last two don't indicate a problem with the network, but it can still be helpful to capture the buffers on the sender and leave the receiver alone. When dealing with the CAN bus, use `candump` as detailed in [debugging.md](../../legacy-docs/can-bus/debugging.md "mention"). When using UDP, it is often simpler to just print the buffer right before it is sent or right as it is received rather than using a tool like [WireShark](https://www.wireshark.org/).&#x20;

## Using ProtobufPal

You'll be greeted with a screen like this:&#x20;

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption><p>The ProtobufPal tool allows for easy and quick debugging</p></figcaption></figure>

To start:&#x20;

* upload or copy-and-paste your Proto code into the left window and click `Validate Proto`
* If you have a buffer you'd like to check, enter it in the bottom windows and click `Decode`
* If you have some data you want to encode, enter it in JSON on the right and click `Encode`

For example, you can enter buffers you see from `candump`, or manually enter your own commands as JSON and send the buffer using `cansend`. Either way, using ProtobufPal allows you to visualize your buffers as easily as JSON would.
