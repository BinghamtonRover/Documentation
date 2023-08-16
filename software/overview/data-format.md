---
description: How we use Protocol Buffers to keep our APIs in sync
---

# Data format

To complete our rover, we need to send data across these devices and layers. This section describes what data is being sent, and the next section describes how it is sent.

Each layer runs in a different environment, with different languages, tools, and packages available. We use [Protocol Buffers](https://protobuf.dev/) (aka, Protobuf) to keep our data in sync despite these challenges. The main advantages of Protobuf, as relevant to our case, are:&#x20;

* **Code generation:** Data is defined in a language-agnostic `.proto` files and Protobuf tools can generate data classes or structs in any language based on those files. This allows us to ensure our data is consistent across layers. If we used, say, JSON, and updated our format on one end, all other ends would need to be manually updated and would throw errors until they do
* **Binary serialization:** Protobuf messages can be serialized directly to binary buffers, which is simpler and more efficient than strings, especially in C++. This also cuts down on unneeded characters such as `{}`, `""`, and `[]`, and identifiers in general
* **Efficient data format:** Protobuf is built on the assumption that only the fields needed for a given message are included. For example, if a `DriveCommand` has `left`, `right`, and `speed` fields, we can provide just the `left` field, and only serialize the data needed. This helps reduce our total network traffic, which is critical for many reasons

Our Protobuf code is hosted in our [Protobuf](https://github.com/BinghamtonRover/Protobuf) repository, which is to be used as a submodule in any other repository as needed. To simplify the process, we submodule it just once in each language, generate the files as needed, and push them to GitHub as a separate repository that can be imported into other repositories. This means we can delegate all Protobuf-related development to one repository, and only one developer needs to install the Protobuf tools and can push the generated files for everyone else.

More information can be found in the [protobuf](../network-details/protobuf/ "mention") document.
