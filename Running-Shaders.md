To run shaders compiled as DXIL, you will need support from the operating system as well as from the driver for your graphics adapter.

At the moment, the [Windows 10 Insider Preview Build 15007](https://blogs.windows.com/windowsexperience/2017/01/12/announcing-windows-10-insider-preview-build-15007-pc-mobile/#XqlQ5FZfXw5WVhpS.97) is able to run DXIL shaders.

Drivers indicate support DXIL by reporting support for Shader Model 6, possibly in experimental mode. To enable support in these cases, the [Developer mode](https://msdn.microsoft.com/windows/uwp/get-started/enable-your-device-for-development) setting must be enabled.

By default, tests will run using the Windows Advanced Rasterization Platform (WARP) adapter. To get the correct version of WARP working, in addition to setting Developer mode, you should install the 'Graphics Tools' optional feature via the Settings app (click the 'Apps' icon, then the 'Manage optional features' link, then 'Add a feature', and select 'Graphics Tools' from the list).

To select the first available adapter that supports D3D12 instead, the parameter /p:"Adapter=*" can be added to the test command line in utils/hct/hcttest.cmd.

Hardware Drivers:

NVIDIA r378 drivers (r378.49 and later) provide experimental mode support for DXIL and shader model 6. This is an early beta version to enable developers to try out DXIL and the new shader model 6 features – Wave Math and int64. Only DXIL version 0.7 (beta) is accepted by the r378 driver.  Experimental mode support for DXIL v1.0 will be provided in a future driver release. 
