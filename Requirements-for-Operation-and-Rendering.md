Using the new compiler for rendering requires the following components:

### Windows 10 and DirectX12:
Support for DXIL was added to Windows 10 via updates last year. Most end user machines should be capable by now.

### Current renderers:
Drivers and renderers that consume DXIL are still at pre-release status so their DXIL interface operates in experimental mode.
Experimental mode requires a build of Windows 15007 or later, such as a recent flight. Instructions for enabling Experimental mode are in the readme.

### Software renderer:
The new WARP12 renderer can be installed as a Windows optional feature (Feature On Demand). Check the readme for installation instructions.

### Hardware GPU renderers:
To use a hardware renderer, please request an experimental mode (pre-release) driver from your friendly IHV.

