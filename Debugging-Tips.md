This page provides some tips on debugging the DirectX Shader Compiler.

## Breaking on Exit

When running console programs from Visual Studio, the console window is closed upon process termination, making it difficult to look out the output.

A simple way to avoid this problem is to set a breakpoint when the process is exiting. From the Breakpoints window, a new breakpoint can be added for `{,,kernel32.dll}ExitProcess` to achieve this effect.

## Native Visualizers

The project has debugger visualizers for clang-level and LLVM-level programs. A symbolic link can be created to point directly to it from the Visual Studio directory. These will get reloaded at the start of every debugging session. It's useful to include both when debugging dxcompiler.dll.

```
mklink "%USERPROFILE%\Documents\Visual Studio 2015\Visualizers\clang.natvis" %HLSL_SRC_DIR%\tools\clang\utils\clang.natvis
mklink "%USERPROFILE%\Documents\Visual Studio 2015\Visualizers\llvm.natvis" %HLSL_SRC_DIR%\utils\llvm.natvis
```

The syntax for .natvis files is documented [here](https://msdn.microsoft.com/en-us/library/jj620914.aspx).

Diagnosing problems with .natvis files is quite easy. You can enable diagnostics in Tools->Options->Debugging->Output Window and select one of the options: off, error, warning or verbose, which will provide additional information concerning the parse failures of the XML or with evaluation failures when looking at data.
