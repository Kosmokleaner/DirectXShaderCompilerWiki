To run HLSL shaders compiled as DXIL requires a current operating system version as well as a recent driver for your graphics adapter.

You will also need to have dxil.dll available during compilation to have the shaders be properly verified for use; this should be available from the Windows SDK under redist\d3d\x64 or redist\d3d\x86. Without dxil.dll present, the compiled shaders won't be usable with drivers, and you will get an error from the Direct3D API when calling CreateComputePipelineState or CreateGraphicsPipelineState.

While support for retail DXIL has been available across Windows 10 for a while now, production drivers expose the new capability via 'experimental mode' which requires the [Windows 10 Creators Update](https://www.microsoft.com/en-us/software-download/windows10?ranMID=24542&ranEAID=TnL5HPStwNw&ranSiteID=TnL5HPStwNw-ydKo1P0j6OJwADi7QUCfLg&tduid=(34190da320062734ab35e1018dc7f8bd)(256380)(2459594)(TnL5HPStwNw-ydKo1P0j6OJwADi7QUCfLg)())
(actually any Windows 10 Insider Preview Build 15007 or later). When compiling a shader for use with experimental mode, dxil.dll need not be present (and may in fact compilation if the shader uses an unrecognized or unsupported feature).

To use a driver in Windows 10 experimental mode, enable the [Developer Mode](https://msdn.microsoft.com/windows/uwp/get-started/enable-your-device-for-development) setting.

### Hardware Drivers:

For the latest on supported hardware drivers, see the project [README.md](https://github.com/Microsoft/DirectXShaderCompiler/blob/master/README.md#running-shaders).

### Software Rendering

In the absence of hardware support, tests will run using the Windows Advanced Rasterization Platform (WARP) adapter. To get the correct version of WARP working, in addition to setting Developer mode, you should install the 'Graphics Tools' optional feature via the Settings app (click the 'Apps' icon, then the 'Manage optional features' link, then 'Add a feature', and select 'Graphics Tools' from the list).

To select the first available adapter that supports D3D12 instead, the parameter /p:"Adapter=*" can be added to the test command line in utils/hct/hcttest.cmd.

