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
