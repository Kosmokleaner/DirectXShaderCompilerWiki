Welcome to the DirectXShaderCompiler wiki!

# DirectX Shader Compiler

The DirectX Shader Compiler is a compiler and related set of tools used to compile High-Level Shader Language (HLSL) programs into DirectX Intermediate Language (DXIL) representation. Applications that make use of DirectX for graphics, games, and computation can use it to generate shader programs.

## Features and Goals
[wiki page](https://github.com/Microsoft/DirectXShaderCompiler/wiki/Features-and-Goals)

## Building Sources
To see how to build the compiler yourself, [check here.]
(https://github.com/Microsoft/DirectXShaderCompiler/wiki/Building-Sources)

## Running Tests
To run the tests that verify the compiler built correctly, [check here](https://github.com/Microsoft/DirectXShaderCompiler/wiki/Runnning-Tests)

## Running Shaders
To compile shaders and samples with this compiler, [check here](https://github.com/Microsoft/DirectXShaderCompiler/wiki/Running Shaders)

## Making Changes

To make contributions, see the [Contributing.md](https://github.com/Microsoft/DirectXShaderCompiler/blob/master/CONTRIBUTING.md) file in this project.

## Documentation

You can find documentation for this project in the docs/ directory. These contain the original LLVM documentation files, as well as two new files worth nothing:

* HLSLChanges.rst: this is the starting point for how this fork diverges from the original llvm/clang sources
* DXIL.rst: this file contains the specification for the DXIL format
* tools/clang/docs/UsingDxc.rst: this file contains a user guide for dxc.exe

## Other Useful Tools

These are recommendations from experience to improve your coding, not official endorsements.

* [Text Macros](https://visualstudiogallery.msdn.microsoft.com/8e2103b6-87cf-4fef-9410-a580c434b602). This adds record/playback for keyboard sequenhces.
* [clang-format plugin for Visual Studio](http://llvm.org/builds/). This is the code standard we are looking to use.

## License

DirectX Shader Compiler is distributed under the terms of the MIT license.

See LICENSE-MIT and COPYRIGHT for details.

## Code of Conduct

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

