The following table provides important information about released versions of the compiler components. The compiler releases as dxcompiler.dll, and can be recognized by the !llvm.ident value produced in DXIL modules. The validator releases as dxil.dll, and can be identified by a runtime query; the rules applies can be identified via DXIL metadata.

| Release | File | File version | Notes |
|---------|------|--------------|-------|
| Windows 10 Creators Update     | dxil.dll | 10.0.15063.0 | no validator version |
| Windows 10 Creators Update SDK | dxcompiler.dll | 10.0.15063.0 | !llvm.ident:dxc 1.0 |
| Windows 10 Fall Creators Update     | dxil.dll | 10.0.16299.15 | validator version 1.1 |
| Windows 10 Fall Creators Update SDK | dxcompiler.dll | 10.0.16299.15 | !llvm.ident:dxc 1.1 |

All these releases have been via the Windows SDKs or PIX tools. PIX releases have a file version of the form 'dxcoob 0.2017.6.0' and the matching identifier would read '!llvm.ident:dxcoob 2017.06'

The following releases are combining bits from GitHub with the OS-produced dxil.dll. The goal is to provide these milestones every month or two until the next major OS and associated SDK ships.

| Release | File | File version | Notes |
|---------|------|--------------|-------|
| Windows 10 19H1 Update SDK | dxil.dll | 10.0.18362.1| 19H1 |
| &nbsp&nbsp&nbsp 2019-07-15 preview | dxc.exe, dxcompiler.dll | 1.4.0.2274 | appveyor build |
| Windows 10 20H1 flight SDK | dxil.dll | 10.0.19xxx.1 | 20H1 |
| &nbsp &nbsp &nbsp 2019-09-12 preview | dxc.exe, dxcompiler.dll | 10.0.19xxx.1 |  appveyor build |


