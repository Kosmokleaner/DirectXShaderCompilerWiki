Welcome to the DirectXShaderCompiler wiki!

# DirectX Shader Compiler

The DirectX Shader Compiler is a compiler and related set of tools used to compile High-Level Shader Language (HLSL) programs into DirectX Intermediate Language (DXIL) representation. Applications that make use of DirectX for graphics, games, and computation can use it to generate shader programs.

## Features and Goals

The starting point of the project is a fork of the [LLVM](http://llvm.org/) and [Clang](http://clang.llvm.org/) projects, modified to accept HLSL and emit a validated container that can be consumed by GPU drivers.

At the moment, the DirectX HLSL Compiler provides the following components:

- dxc.exe, a command-line tool that can compile shader model 6 HLSL programs

- dxcompiler.dll, a DLL providing a componentized compiler, assembler, disassembler, and validator

- various other tools based on the above components

The DirectX Shader Compiler is currently in preview stage but is expected to be finalized in the next few months. The Microsoft Windows SDK releases will include a supported version of the compiler and validator.

The goal of the project is to allow the broader community of shader developers to contribute to the language and representation of shader programs, maintaining the principles of compatibility and supportability for the platform. It's currently in active development across two axes: language evolution (with no impact to DXIL representation), and surfacing hardware capabilities (with impact to DXIL, and thus requiring coordination with GPU implementations).

## Building Sources

Before you build, you will need to have some additional software installed.

* [Git](http://git-scm.com/downloads).
* [Visual Studio 2015](https://www.visualstudio.com/downloads). Update 3 is the supported version. This will install the Windows Development Kit as a side effect. We also need the common tools for visual C++ to get the atl headers (e.g. atlbase.h). In the install options, make sure the following options are checked:
    * Windows 10 SDK (10.0.10240.0)
    * Common Tools for Visual C++ 2015
* [Windows 10 SDK](https://developer.microsoft.com/en-US/windows/downloads/windows-10-sdk). This is needed to build tests that reference the D3D12 runtime.
* [Windows Driver Kit](https://developer.microsoft.com/en-us/windows/hardware/windows-driver-kit). Or download and install the [Windows Driver Kit 8.1 Update 1](http://www.microsoft.com/en-us/download/details.aspx?id=42273), no need to download and install tests. This will target your common Windows Kits path, on a 64-bit machine this will likely be C:\Program Files (x86)\Windows Kits\8.1. WDK for Windows 10 should also work. This is currently needed to run TAEF tests and build with the TAEF framework.
* [CMake](https://cmake.org/files/v3.4/cmake-3.4.3-win32-x86.exe). Version 3.4.3 is the supported version. You need not change your PATH variable during installation.
* [Python](https://www.python.org/downloads/). Version 2.7.x is required, 3.x might work but it's not officially supported. You need not change your PATH variable during installation.

To setup the build environment run the `utils\hct\hctstart.cmd` script passing the path to the source and build directories. For example:

    git clone <DirectXShaderCompiler repo> C:\DirectXShaderCompiler
    cd C:\DirectXShaderCompiler
    utils\hct\hctstart.cmd C:\DirectXShaderCompiler C:\DirectXShaderCompiler.bin

To create a shortcut to the build environment with the default build directory, double-click on the `utils\hct\hctshortcut.js` file.

To build, open the HLSL Console and run this command.

    hctbuild

You can also clean, build and run tests with this command.

    hctcheckin 

## Running Tests

To run tests, open the HLSL Console and run this command after a successful build.

    hcttest

Some tests will run shaders and verify their behavior. These tests also involve a driver that can run these execute these shaders. See the next section on how this should be currently set up.

## Running Shaders

To run shaders compiled as DXIL, you will need support from the operating system as well as from the driver for your graphics adapter.

At the moment, the [Windows 10 Insider Preview Build 15007](https://blogs.windows.com/windowsexperience/2017/01/12/announcing-windows-10-insider-preview-build-15007-pc-mobile/#XqlQ5FZfXw5WVhpS.97) is able to run DXIL shaders.

Drivers indicate support DXIL by reporting support for Shader Model 6, possibly in experimental mode. To enable support in these cases, the [Developer mode](https://msdn.microsoft.com/windows/uwp/get-started/enable-your-device-for-development) setting must be enabled.

By default, tests will run using the Windows Advanced Rasterization Platform (WARP) adapter. To select the first available adapter that supports D3D12 instead, the parameter /p:"Adapter=*" can be added to the test command line in utils/hct/hcttest.cmd.

## Making Changes

To make contributions, see the CONTRIBUTING.md file in this project.

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

