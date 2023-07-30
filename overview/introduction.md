# Introduction

Our rover is a very complex system, which means our code has the complexity to match. Complexity means layers of abstraction and delegation of responsibilities. Even if we _could_ have a single million-line file, our code needs to run on multiple devices in different environments, so we need to choose the best platform, language, framework, and tools for each scenario.&#x20;

The next pages describe the different devices on the rover and how they interact on a basic level. The following sections provide more details about how each system works, fundamental concepts, and how to start writing code.&#x20;

**These docs describe the high-level design and do not describe the exact API, functions, or lines of code.** Code should be documented in-source using comments that start with `///` , which will then be picked up by our tooling ([Doxygen](https://www.doxygen.nl/) and [DartDocs](https://dart.dev/language/comments#documentation-comments)) and extracted elsewhere. See the page on [code documentation](documenting-code.md) for more information.&#x20;
