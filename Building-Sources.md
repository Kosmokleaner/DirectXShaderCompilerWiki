# Getting Ready to Build

Before you build, you will need to have some additional software installed. This section provides the most streamlined configuration, based on Visual Studio 2017 - other configurations are available.

* [Git](http://git-scm.com/downloads).
* [Visual Studio 2017](https://www.visualstudio.com/downloads). Select the following workloads: Universal Windows Platform Development and Desktop Development with C++.
* [Python](https://www.python.org/downloads/). Version 3.x is recommended. You need not change your PATH variable during installation.

After cloning the project, you can set up a build environment shortcut by double-clicking the `utils\hct\hctshortcut.js` file. This will create a shortcut on your desktop with a default configuration.

## Building

Start by opening the HLSL console from the desktop shortcut or via `hctstart`.

Tests are built using the TAEF framework. Unless you already have this, you should run the script at `utils\hct\hctgettaef.py` from your build environment before you start building to download and unzip them as an external dependency. You should only need to do this once.

From an HLSL Console, run this command to build a solution and all projects.

    hctbuild

You can also run tests with this command.

    hcttest

After you have built a solution, you can open it in Visual Studio by running `hctvs`.

# Other Configurations

## Using hctstart

To specify build environment options, run the `utils\hct\hctstart.cmd` script passing the path to the source and build directories. For example, if you cloned with `git clone https://github.com/Microsoft/DirectXShaderCompiler.git C:\DirectXShaderCompiler`, you can run the following to have the output in `C:\DirectXShaderCompiler.bin`.

    utils\hct\hctstart.cmd C:\DirectXShaderCompiler C:\DirectXShaderCompiler.bin

## Using SDK and WDK installations

Visual Studio 2017 and Visual Studio 2015 Update 3 both include the ability to install a supported Windows SDK.
The [Windows 10 SDK](https://developer.microsoft.com/en-US/windows/downloads/windows-10-sdk) is needed to build tests that reference the D3D12 runtime. You may get this as part of installing/updating Visual Studio.

Install TAEF and some needed headers by installing the [Windows Driver Kit](https://developer.microsoft.com/en-us/windows/hardware/windows-driver-kit). No need to download and install tests. This is used to build and run tests. Do **not** select the WDK Visual Studio plugin or cmake will fail to find your compiler. 

## Using Visual Studio 2015

To build from the command line follow the normal build steps, but pass `-vs2015` as a parameter to `hctbuild`.

The supported version of Visual Studio 2015 is Update 3. In the install options, make sure the following options are checked:

 * Windows 10 SDK (version 14393)
 * Common Tools for Visual C++ 2015

Visual Studio 2017 provides CMake support, but you'll need to install version [3.4.3](https://cmake.org/files/v3.4/cmake-3.4.3-win32-x86.exe) or [3.7.2](https://cmake.org/files/v3.7/cmake-3.7.2-win32-x86.msi) if using Visual Studio 2015. You need not change your PATH variable during installation. 3.7.2 is needed to build with Visual Studio 2017.

## Using Visual Studio 2017

You can build with vs2017 either on the command line or using the integrated [CMake support](https://blogs.msdn.microsoft.com/vcblog/2016/11/16/cmake-support-in-visual-studio-the-visual-studio-2017-rc-update/).

To build from the command line follow the normal build steps, but pass `-vs2017` as a parameter to `hctbuild`.

To build using the integrated CMake support, simply start Visual Studio and open the folder where you have the source. From the CMake menu select "Build CMakeLists.txt"

By default the binaries will be built in %LOCALAPPDATA%\CMakeBuild\DirectXShaderCompiler\build\{build-flavor}.
The build location can be changed by editing the `CMakeSettings.json` file.

You can then use the build directory in the `hctstart` script to test the build. For example,

    hctstart C:\source\DirectXShaderCompiler %LOCALAPPDATA%\CMakeBuild\DirectXShaderCompiler\build\x64-Debug

## Using Visual Studio 2019

To build from the command line follow the normal build steps, but pass `-vs2019` as a parameter to `hctbuild`.

Visual Studio 2019 may require a more recent version of cmake.

## Using Ninja

To build with Ninja, please make sure that you have `ninja` and `cl` in your `%PATH%`.
`ninja` can be installed from [here](https://github.com/ninja-build/ninja/releases);
`cl` should already be installed together with Visual Studio and can be exported to `%PATH%` via the `vcvars*.bat` script in Visual Studio's VC build directory.

To configure cmake with Ninja generator,

    hctbuild -s -ninja

To build with Ninja, go to the binary directory and run `ninja` directly or

    hctbuild -b -ninja

If you use Ninja to build the project, please make sure to supply `-ninja` to `hcttest` for testing.
