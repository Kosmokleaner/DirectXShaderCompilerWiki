To run HLSL shaders compiled as DXIL requires a current operating system version as well as a recent driver for your graphics adapter.

You will also need to have dxil.dll available during compilation to have the shaders be properly verified for use; this should be available from the latest [github releases](https://github.com/microsoft/DirectXShaderCompiler/releases) or the Windows SDK under redist\d3d\x64 or redist\d3d\x86. Without dxil.dll present, the compiled shaders won't be usable with drivers, and you will get an error from the Direct3D API when calling CreateComputePipelineState or CreateGraphicsPipelineState.

If a shader is not verified and signed with dxil.dll, whether because the library isn't found or because the shader includes features that dxil.dll doesn't recognize yet, using it in a pipeline requires enabling [experimental shader models](https://docs.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-d3d12enableexperimentalfeatures) ([see example](https://github.com/microsoft/DirectXShaderCompiler/blob/0ec5839e5ebb5bcb4eaa19164b441f570622365d/tools/dxexp/dxexp.cpp#L319)) and [Developer Mode](https://msdn.microsoft.com/windows/uwp/get-started/enable-your-device-for-development) on Windows 10 build 15007 or newer.

### Hardware Drivers:

For the latest on supported hardware drivers, see the project [README.md](https://github.com/Microsoft/DirectXShaderCompiler/blob/master/README.md#running-shaders).

### Software Rendering

In the absence of hardware support, tests will run using the Windows Advanced Rasterization Platform (WARP) adapter. To get the correct version of WARP working, in addition to setting Developer mode, you should install the 'Graphics Tools' optional feature via the Settings app (click the 'Apps' icon, then the 'Manage optional features' link, then 'Add a feature', and select 'Graphics Tools' from the list).

To select the first available adapter that supports D3D12 instead, the parameter "-adapter=*" can be added to the test command line in utils/hct/hcttest.cmd.

