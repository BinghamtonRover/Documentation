# Software Documentation

These pages document the Software Team's layout, structure, and best practices. Specifically, these pages will walk you through the structure of the rover, where to find code, and where to find documentation for the code.&#x20;

**These docs describe the high-level design and do not describe the exact API, functions, or lines of code.** Code should be documented in-source using comments that start with `///` , which will then be picked up by our tooling ([Doxygen](https://www.doxygen.nl/) and [DartDocs](https://dart.dev/language/comments#documentation-comments)) and extracted elsewhere. See the page on [code documentation](overview/documenting-code.md) for more information.&#x20;

Our rover is a very complex system, which means our code has the complexity to match. Complexity means layers of abstraction and delegation of responsibilities. Even if we _could_ have a single million-line file, our code needs to run on multiple devices in different environments, so we need to choose the best platform, language, framework, and tools for each scenario.&#x20;

The [Broken link](broken-reference "mention") section provides high-level details of the entire codebase, with later sections providing more details. Please read the entire Overview section to learn about the different parts of the rover, and, using that information, read through as much of the relevant docs for your tasks as possible. This will help give you a better context when writing code and help you ask more focused questions that go beyond "how does this work?"
