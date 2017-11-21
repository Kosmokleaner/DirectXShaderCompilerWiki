Shader Model 6.1 is the first shader model that can be compiled with the DirectX Shader Compiler.

The main additions are as follows:

* [[SV_ViewID]]. This feature enables instancing of the graphics pipeline by "view", in a manner that is orthogonal to draw instancing.
* [[SV_Barycentrics]]. This is a new system-generated value available in pixel shaders, used for example to perform interpolation over small or unaligned values like a few bits from a 32-bit value.

