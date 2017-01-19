# DirectX Shader Compiler

The DirectX Shader Compiler project includes a compiler and related tools used to compile High-Level Shader Language (HLSL) programs into DirectX Intermediate Language (DXIL) representation. Applications that make use of DirectX for graphics, games, and computation can use it to generate shader programs.

## Features and Goals

The starting point of the project is a fork of the [LLVM](http://llvm.org/) and [Clang](http://clang.llvm.org/) projects, modified to accept HLSL and emit a validated container that can be consumed by GPU drivers.

At the moment, the DirectX HLSL Compiler provides the following components:

- dxc.exe, a command-line tool that can compile shader model 6 HLSL programs

- dxcompiler.dll, a DLL providing a componentized compiler, assembler, disassembler, and validator

- various other tools based on the above components

The DirectX Shader Compiler is currently in preview stage but is expected to be finalized in the next few months. The Microsoft Windows SDK releases will include a supported version of the compiler and validator.

The goal of the project is to allow the broader community of shader developers to contribute to the language and representation of shader programs, maintaining the principles of compatibility and supportability for the platform. It's currently in active development across two axes: language evolution (with no impact to DXIL representation), and surfacing hardware capabilities (with impact to DXIL, and thus requiring coordination with GPU implementations).

## Getting Things Done

- [[Build the compiler|Building-Sources]].
- [[Run the tests|Running-Tests]] that verify the compiler built correctly.
- [[Compile shaders and samples|Running Shaders]] with your new compiler.

## Making Changes

To make contributions, see the [Contributing.md](https://github.com/Microsoft/DirectXShaderCompiler/blob/master/CONTRIBUTING.md) file in this project.

## Documentation

You can find documentation for this project in the docs/ directory. These contain the original LLVM documentation files, as well as some new files worth noting:

* [HLSLChanges.rst](https://github.com/Microsoft/DirectXShaderCompiler/blob/master/docs/HLSLChanges.rst):
 this is the starting point for how this fork diverges from the original llvm/clang sources
* [DXIL.rst](https://github.com/Microsoft/DirectXShaderCompiler/blob/master/docs/DXIL.rst):
 this file contains the specification for the DXIL format
* [tools/clang/docs/UsingDxc.rst](https://github.com/Microsoft/DirectXShaderCompiler/blob/master/tools/clang/docs/UsingDxc.rst):
 this file contains a user guide for dxc.exe

## License

DirectX Shader Compiler is distributed under the terms of the MIT license.

See [LICENSE-MIT](https://github.com/Microsoft/DirectXShaderCompiler/blob/master/LICENSE-MIT)
and [COPYRIGHT](https://github.com/Microsoft/DirectXShaderCompiler/blob/master/COPYRIGHT) for details.

## Code of Conduct

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

