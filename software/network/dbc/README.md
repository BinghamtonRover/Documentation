---
description: How we send and receive data over the CAN bus network
---

# CAN Database (DBC)

## Introduction

A CAN Database (DBC) is a file which specifies how messages and signals are encoded and decoded over a CAN bus. CAN DBC files are designed to make sure each message takes the absolute minimum size possible, down to the bit level. CAN bus payloads, which for CAN 2.0, is 8 bytes, may seem small, but when working on an embedded network, it can be more than enough for most messages.

CAN DBC differs from other formats such as Protobuf, where instead of just specifying variables and their types, you have to specify the start and end bits, the size, range, etc.

## Why Not Protobuf?

Protobuf is not designed for small sized embedded networks. There is a significant amount of byte overhead which makes it very challenging to send over CAN. For instance, a message with just one boolean included will take up 2 bytes, where realistically you only need 1 bit for a boolean. With floating points, there are always 4 bytes, even if they could be represented as an integer. Additionally, 4 bytes is often unnecessary for most use cases. For instance, the throttle for drive is a float between 0 and 1. For a small range of numbers, using a 32 bit floating point wastes a ton of data, since floating points are meant for an extremely wide range of numbers. By using a 10 bit integer, and scaling it down by 0.0001, you can achieve the same range with less data.

## DBC Concepts

### Signal

A signal is an individual data value. This is equivalent to a variable in code.

In a DBC file, each signal has the following information defined:
1. Starting bit
2. Endian (Motorola or Intel)
3. Length
4. Scale factor
5. Range (min and max)

### Message

A message is just like how it is with Protobuf, it's a container of data. A message will have several signals representing the data that it carries. The length of a message is rounded up to the nearest byte.
