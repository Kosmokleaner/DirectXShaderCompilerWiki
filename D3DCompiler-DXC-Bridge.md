The d3dcompiler_dxc_bridge.dll is a DLL that will forward many of the D3DCompiler API calls into DirectX Shader Compiler calls in dxcompiler. It can be used to recompile the runtime-compiled shaders of an app into shader model 6 without having to rebuild it or modify any of its source files.

# Usage

The use the library, copy d3dcompiler_dxc_bridge.dll next to the app you're interested in, and rename it d3dcompiler_47.dll. You should also copy dxcompiler.dll and dxil.dll next to the app, so the compiler and validator can be found.

# Gotchas

d3dcompiler_47.dll is the most common DLL these days, but an app or game might be trying to use a prior version, such as d3dcompiler_43.dll. Make sure you rename to the name that the app is using. If necessary, you can have more than one copy of the file with different names.

Some flags aren't implemented because they're not applicable or supported (D3DCOMPILE_ENABLE_BACKWARDS_COMPATIBILITY, D3DCOMPILE_ENABLE_STRICTNESS, D3DCOMPILE_PARTIAL_PRECISION). These are simply ignored in the bridge.

The include handler functionality in dxcompiler has a significantly different design and can't really be reimplemented. The dxcompiler version of the handler has no memory of the history of calls and will return file-not-found if a file isn't found with a specific name. The d3dcompiler version instead had to track open/close pairs and reimplement rules for local and system includes. Because of this limitation, there is no support for using a custom include handler (see [this thread](https://github.com/Microsoft/DirectXShaderCompiler/issues/161) for more information).

It's possible that we're missing some API call you care deeply about - if so, please file an Issue and we can discuss how to resolve your case.

# Sources

The sources are available under tools\clang\tools\d3dcomp.
