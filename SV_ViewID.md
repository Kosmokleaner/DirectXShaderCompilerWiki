# Overview

This feature enables instancing of the graphics pipeline by "view", in a manner that is orthogonal to draw instancing.  Looping of view instances can happen anywhere from before draw instancing to late in the graphics pipeline depending on the sophistication of the implementation. Meanwhile applications can write one codepath for driving multiple views that can target the breadth of hardware.   

Graphics shaders can read system value SV_ViewID [0..view instance count] identifying the current view. An obvious way for an app to use the feature is for the shader stage feeding the rasterizer to generate SV_Position position as a function of SV_ViewID, so a single draw call can send geometry to multiple output surface locations with different projections, such as left and right eye for stereo rendering.

## SV_ViewID

Graphics shaders in shader model 6.1+ can input unsigned 32-bit integer system value SV_ViewID identifying the current view.  These inputs don't appear in shader input or output signatures for the purpose of shader linkage, and shaders can't output them (as system values).  

If a shader references SV_ViewID, that reference and its dependencies logically/implicitly become instanced by ViewInstanceCount.   

If the Pixel Shader inputs SV_ViewID, one of the input vertex data scalars is reserved (taken away from the amount of data the application can put in its vertices) to allow some implementations to pass the SV_ViewID through vertex data.  Applications don't need to bother outputting SV_ViewID to the Pixel Shader from the upstream shader – only the implementations that need to will do it.

SV_ViewID is mapped to the dx.op.viewID() intrinsic in DXIL. 

## View Dependent Vertex Storage 

Any shader output that is a function of SV_ViewID implicitly costs scalars contributing, subject to signature packing constraints, towards the 128 scalar limit on vertex size between any two shader stages.  This limit is enforced for uniformity. 

There is nothing the application has to indicate in its shader output declaration about which attributes are view dependent.  The HLSL compiler does annotate, in the bytecode it generates, which outputs could have been influenced by an SV_ViewID reference based on code flow.  Regardless of whether the actual computed values end up being view varying or not, any output with a possible SV_ViewID dependency is assumed to vary per view by implementations.  This annotation can be leveraged by runtimes as needed.

The HLSL compiler generates metadata in shader bytecode to assist with validation. There are three components to the metadata: 

(1) A bit for every scalar output of a shader indicating if it could be influenced by a reference to ViewID in that shader 

(2) A bit vector that describes for every scalar output from a shader which scalar inputs influence it

(3) Shader input/output/pc arrays have a bit mask indicating which components are dynamically indexed 

Below are some important details on the HLSL compiler analysis: 

(1) The compiler analyzes hull shader stages independently; i.e., values in one stage do not affect values in the other stage. The analysis resolves that patch constant stage outputs depend on control point (main entry) inputs; the runtime/driver does not need to do this. 

(2) UAV accesses do not propagate dependency on ViewID as it is unlikely that an implementation is willing to replicate a UAV; that is, loads from UAVs are never dependent on ViewID.

(3) The current implementation of ViewID state analysis in the HLSL compiler is somewhat conservative. There is no context-sensitive analysis of loads and stores for indexable registers. As such, any ViewID-dependent store to such a register, makes all loads from this variable depend on ViewID in the entire shader. Future versions of the compiler may implement a more precise, context-sensitive analysis of loads and stores. 
