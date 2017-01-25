Using the new compiler for rendering requires the following components:

### Windows 10 and DirectX12:
Support for DXIL was released for end users in updates last year.

### Current renderers (drivers, etc. that consume DXIL):
These are still at pre-release status so their DXIL interface operates in experimental mode.
Experimental mode requires a build of Windows 15007 or later, such as a recent flight.

### Software renderer:
The new WARP12 renderer can be installed as a Windows optional feature (Feature On Demand).

### Hardware GPU renderers:
To use a hardware renderer, please request an experimental mode (pre-release) driver from your friendly IHV.

Instructions for these installations are in the Readme.
