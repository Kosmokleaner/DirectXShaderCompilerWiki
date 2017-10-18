The following table provides important information about released versions of the compiler components. The compiler releases as dxcompiler.dll, and can be recognized by the !llvm.ident value produced in DXIL modules. The validator releases as dxil.dll, and can be identified by a runtime query; the rules applies can be identified via DXIL metadata.

| Release | File | File version | Notes |
|---------|------|--------------|-------|
| Windows 10 Creators Update     | dxil.dll | 10.0.15063.0 | no validator version |
| Windows 10 Creators Update SDK | dxcompiler.dll | 10.0.15063.0 | !llvm.ident:dxc 1.0 |
| Windows 10 Fall Creators Update     | dxil.dll | 10.0.16299.15 | validator version 1.1 |
| Windows 10 Fall Creators Update SDK | dxcompiler.dll | 10.0.16299.15 | !llvm.ident:dxc 1.1 |

Note that so far, all existing releases have been via the Windows SDKs or PIX tools. PIX releases have a file version of the form 'dxcoob 0.2017.6.0' and the matching identifier would read '!llvm.ident:dxcoob 2017.06'
