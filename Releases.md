The following table provides important information about released versions of the compiler components. The compiler releases as dxcompiler.dll, and can be recognized by the !llvm.ident value produced in DXIL modules. The validator releases as dxil.dll, and can be identified by a runtime query; the rules applies can be identified via DXIL metadata.

The latest release including the dxcompiler.dll library, the dxc.exe executable, and the dxil.dll validator library can be downloaded from the GitHub project [releases](https://github.com/microsoft/DirectXShaderCompiler/releases) tab. 

| Release | Files | Version |
|---------|------|--------------|
| June 2021 | dxc.exe, dxcompiler.dll, dxil.dll | 1.6.2106 |
| April 2021 | dxc.exe, dxcompiler.dll, dxil.dll | 1.6.2104 |
| October 2020 | dxc.exe, dxcompiler.dll, dxil.dll | 1.5.2010 |
| March 2020 pre-release | dxc.exe, dxcompiler.dll, dxil.dll | 1.5.2003 |

The compiler binaries are also shipped along with the Windows SDK. These releases lag behind the GitHub releases considerably, but allow shader compilation with Visual Studio without any additional effort.

Information on the latest Windows SDK release:

| Release | File | File version | Notes |
|---------|------|--------------|-------| 
| Windows 10, version 21H1  | dxil.dll | 10.0.19041.685 | validator v1.5 |
| Windows 10, version 21H1 SDK  | dxcompiler.dll | 10.0.19041.685 | !llvm.ident:dxc 1.5 |


The compiler version can be accessed from shader code using the [[Predefined Version Macros]] `__DXC_VERSION_MAJOR`, `__DXC_VERSION_MINOR`, `__DXC_VERSION_RELEASE`, `__DXC_VERSION_COMMITS` representing the 4-part version string produced by `dxc -?` or `dxc --version`
