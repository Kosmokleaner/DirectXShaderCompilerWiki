The HLSL shader model is a versioning approach indicating which new features are added to the language. Each level allows an application or game to target a well-known set of functionality for development, and allows hardware and driver developers to target that same description for support.

The currently targetted shader model can be accessed from within the shader code using the [[Predefined Version Macros]] `__SHADER_TARGET_MAJOR` and `__SHADER_TARGET_MINOR`.

# Versions

Shader model versions gradually introduce new processing stages, relax constraints and introduce a superset of capabilities.

* [Shader Model 1](https://msdn.microsoft.com/en-us/library/windows/desktop/bb509654(v=vs.85).aspx). This was the first shader model created in DirectX. It introduced vertex and pixel shaders to the first implementation of the programmable pipeline.
* [Shader Model 2](https://msdn.microsoft.com/en-us/library/windows/desktop/bb509655(v=vs.85).aspx). Adds new intrinsics and increases limits on registers and instructions.
* [Shader Model 3](https://msdn.microsoft.com/en-us/library/windows/desktop/bb509656(v=vs.85).aspx). Adds new intrinsics and increases limits on registers and instructions.
* [Shader Model 4](https://msdn.microsoft.com/en-us/library/windows/desktop/bb509657(v=vs.85).aspx). This is a superset of the capabilities in Shader Model 3, except that Shader Model 4 doesn't support the features in Shader Model 1. It has been designed using a common-shader core that gives a common set of features to all programmable shaders, which are only programmable using HLSL. It adds new shader profiles to target geometry shaders.
* [Shader Model 5](https://msdn.microsoft.com/en-us/library/windows/desktop/ff471356(v=vs.85).aspx). This is a superset of shader model 4 and adds new resources, compute shaders and tessellation.
* [Shader Model 5.1](https://msdn.microsoft.com/en-us/library/windows/desktop/dn933277(v=vs.85).aspx). This is functionally very similar to Shader Model 5; the main change is more flexibility in resource selection by allowing indexing of arrays of descriptors from within a shader.
* [[Shader Model 6.0]]. This is a superset of shader model 5.1 with some deprecated language elements and with the addition of wave intrinsics and 64-bit integers for arithmetic.
* [[Shader Model 6.1]]. This is a superset of shader model 6.0 that adds support for SV_ViewID, barycentric semantics and the GetAttributeAtVertex intrinsic.
* [[Shader Model 6.2]]. Adds support for float16 (as opposed to minfloat16) and denorm mode selection.
* [[Shader Model 6.3]]. Adds support for DirectX Raytracing (DXR), including libraries and linking.
* [[Shader Model 6.4]]. Adds Variable Rate Shading, low-precision packed dot product intrinsics, and support for library sub-objects to simplify raytracing.
* [[Shader Model 6.5]]. Adds support for DXR 1.1, Sampler Feedback, Mesh and Amplification shaders, and additional Wave Intrinsics.
* [[Shader Model 6.6]]. Adds support for new atomic operations (inc. 64-bit), dynamic resources, IsHelperLane(), derivatives in compute, mesh, and amplification shaders, pack/unpack, WaveSize, and Raytracing Payload Access Qualifiers.
