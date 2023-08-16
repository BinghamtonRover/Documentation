# Generating code

## Introduction

Generating code may be straightforward but involves downloading software, updating your compiler, and getting the command just write for your language. To ease this pain:&#x20;

* CLI scripts are pushed to the Protobuf repository with the exact command for each language
* Generated code is pushed to shared repositories (by language)

Together, this means that only a handful of people need to be educated on how to generate code, and they can push it for others to use without requiring everyone to download Protobuf.

## Getting set up

You can find the Protobuf compiler (`protoc`) [here](https://github.com/protocolbuffers/protobuf/releases/latest). The compiler only supports a few languages by default and needs special setup for some languages

### Dart

To setup your Dart installation, install the [`protoc-plugin`](https://pub.dev/packages/protoc\_plugin) package to your **computer**:

```bash
dart pub global activate protoc_plugin
```

The compiled code will import the [`protobuf`](https://pub.dev/packages/protobuf) package, which you need to add to each **project**:

```bash
dart pub add protobuf
```

### Nanopb (C++)

The `protoc` compiler has C++ support out-of-the-box, but it's not fit for microcontrollers like Arduino. Instead, we use another toolchain called [`nanopb`](https://jpa.kapsi.fi/nanopb/), which both provides the compiler and the libraries that it will import. The libraries are already included in the [`Firmware-Utilities`](https://github.com/BinghamtonRover/Firmware-Utilities) repository, but you'll need to install the Python-based compiler yourself:

```bash
pip install nanopb
```

Nanopb has many differences compared to the standard Protobuf, especially when comparing C++ to more modern languages, but so long as you follow the [#guidelines](working-with-proto-files.md#guidelines "mention"), the code should be compatible.

## Compiling code

Recall that we push our code to a shared repository for each language ([#generated-code](./#generated-code "mention")). For the firmware, we need to generate code in each repository as well.

### Git Submodule

We first need to include the Protobuf code in each repository so we can compile it. Instead of cloning the Protobuf repository, include it as a _submodule_, a nested library.&#x20;

```bash
git submodule add https://github.com/BinghamtonRover/Protobuf
```

A submodule is a whole other repository, so to make changes to Proto files, `cd` into it, make a new branch and test your changes. When you're ready push and open a PR. Whenever you pull or change commits, you'll need to `git add` that to the parent repository as well.

### Using our build scripts

Once you have the desired compiler in your path, you can use our build scripts in the [Protobuf](https://github.com/BinghamtonRover/Protobuf) repository to handle the build for you. These scripts are written in a language you'll already have.&#x20;

#### Dart

At the root of your Dart project, the following comand compiles all Protobuf files

```bash
dart gen_dart.dart
```

#### Nanopb (C++)

In a firmware repository, first submodule the Proto files as explained above, then run:

```bash
python generate_arduino.py -s SKETCH_NAME -p PROTO_NAME
```

Unlike Dart, Arduino will compile all available source code. To avoid compiling code we do not need, use this command with just the name of the Protobuf file you need. `core.proto` is already compiled in `firmware-utilities`.&#x20;

## More resources

* [The Protobuf Dart tutorial](https://protobuf.dev/getting-started/darttutorial/)
* [The Nanopb docs](https://jpa.kapsi.fi/nanopb/docs/index.html)
* [The Protobuf Wire Format](https://protobuf.dev/programming-guides/encoding/)
