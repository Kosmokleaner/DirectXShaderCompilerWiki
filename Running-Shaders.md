To run shaders compiled as DXIL, you will need support from the operating system as well as from the driver for your graphics adapter.
This [link](https://github.com/Microsoft/DirectXShaderCompiler/wiki/Requirements-for-Operation) lists the components required for operation.

While support for retail DXIL is available across Windows 10 today, prototype drivers use Experimental Mode which requires the [Windows 10 Insider Preview Build 15007](https://blogs.windows.com/windowsexperience/2017/01/12/announcing-windows-10-insider-preview-build-15007-pc-mobile/#XqlQ5FZfXw5WVhpS.97) or later.

Drivers indicate support DXIL by reporting support for Shader Model 6, possibly in experimental mode. To enable support in these cases, the [Developer mode](https://msdn.microsoft.com/windows/uwp/get-started/enable-your-device-for-development) setting must be enabled.

###Hardware Drivers:

The following vendors provide drivers with hardware GPU support for DXIL:

NVIDIA r378 drivers (r378.49 and later) provide experimental mode support for DXIL and shader model 6. This is an early beta version to enable developers to try out DXIL and the new shader model 6 features – Wave Math and int64. Only DXIL version 0.7 (beta) is accepted by the r378 driver.  Experimental mode support for DXIL v1.0 will be provided in a future driver release.

AMD released its first driver with experimental mode support for DXIL and Shader Model 6: [Radeon Software Crimson ReLive Edition 17.3.1](https://community.amd.com/docs/DOC-1771). It uses and requires DXIL v1.0. Using this driver, all of the execution tests pass except for the 2 wave intrinsics tests. This capability is also provided in the update, [Radeon Software Crimson ReLive Edition 17.3.2](http://support.amd.com/en-us/kb-articles/Pages/Radeon-Software-Crimson-ReLive-Edition-17.3.2-Release-Notes.aspx).

A later driver should have fixes for the wave routines.

###Software Rendering

In the absence of hardware support, tests will run using the Windows Advanced Rasterization Platform (WARP) adapter. To get the correct version of WARP working, in addition to setting Developer mode, you should install the 'Graphics Tools' optional feature via the Settings app (click the 'Apps' icon, then the 'Manage optional features' link, then 'Add a feature', and select 'Graphics Tools' from the list).

To select the first available adapter that supports D3D12 instead, the parameter /p:"Adapter=*" can be added to the test command line in utils/hct/hcttest.cmd.

