To run HLSL shaders compiled as DXIL requires a current operating system version as well as a recent driver for your graphics adapter.

While support for retail DXIL has been available across Windows 10 for a while now, production drivers expose the new capability via 'experimental mode' which requires the [Windows 10 Creators Update](https://www.microsoft.com/en-us/software-download/windows10?ranMID=24542&ranEAID=TnL5HPStwNw&ranSiteID=TnL5HPStwNw-ydKo1P0j6OJwADi7QUCfLg&tduid=(34190da320062734ab35e1018dc7f8bd)(256380)(2459594)(TnL5HPStwNw-ydKo1P0j6OJwADi7QUCfLg)())
(actually any Windows 10 Insider Preview Build 15007 or later).

Drivers indicate support DXIL by reporting support for Shader Model 6, possibly in experimental mode. To enable support in these cases, the [Developer Mode setting](https://msdn.microsoft.com/windows/uwp/get-started/enable-your-device-for-development) must be enabled.

### Hardware Drivers:

The following vendors provide drivers with hardware GPU support for DXIL:

NVIDIA's new r381 drivers (r381.65 and later) provide experimental mode support for DXIL 1.0 and Shader Model 6.0. This feature is considered beta at this time and intended to enable developers to try out DXIL and the new Shader Model 6.0 features -- Wave Math and Int64. Here are the [release notes.](http://us.download.nvidia.com/Windows/381.65/381.65-win10-win8-win7-desktop-release-notes.pdf), and a [download link](http://uk.download.nvidia.com/Windows/381.65/381.65-desktop-win10-64bit-international-whql.exe).

AMD's latest driver with support for DXIL 1.0 and Shader Model 6 in experimental mode is: [Radeon Software Crimson ReLive Edition 17.4.2](http://support.amd.com/en-us/kb-articles/Pages/Radeon-Software-Crimson-ReLive-Edition-17.4.2-Release-Notes.aspx).

### Software Rendering

In the absence of hardware support, tests will run using the Windows Advanced Rasterization Platform (WARP) adapter. To get the correct version of WARP working, in addition to setting Developer mode, you should install the 'Graphics Tools' optional feature via the Settings app (click the 'Apps' icon, then the 'Manage optional features' link, then 'Add a feature', and select 'Graphics Tools' from the list).

To select the first available adapter that supports D3D12 instead, the parameter /p:"Adapter=*" can be added to the test command line in utils/hct/hcttest.cmd.

