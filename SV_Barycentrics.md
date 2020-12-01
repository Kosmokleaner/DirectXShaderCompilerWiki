# Overview
Barycentric coordinates are a common method for defining locations within a geometric primitive such as a triangle or line. This document describes a mechanism for pixel shaders to read barycentric coordinates of the current pixel relative to the containing primitive. These can then be used for custom attribute interpolation, such as higher-order interpolation schemes, creative attribute unpacking, or allowing per-vertex attributes to be specified and interpolated at other precisions. Effectively, barycentric coordinates cleanly isolate the process of mapping the current pixel to a location within the primitive being rasterized, and associated fixed-function logic for interpolating the attribute at that location, which can then be done on the shader processor. We consider this an important direction for future evolution of the graphics pipeline.
Two relatively orthogonal mechanisms are described, accessing the barycentric weights, and accessing attributes at the primitive vertices to be interpolated with those weights.

## System Generated Barycentric Coordinates
Pixel shaders can access barycentric weights in their input attribute declaration via the SV_Barycentrics semantic system value, following the standard attribute declaration syntax:

    float4 PSMain(float3 baryWeights : SV_Barycentrics) : SV_Target {
       ...
    }

Attributes declared with the SV_Barycentrics system-generated value must be a vector of three 32-bit floating point numbers, representing the barycentric weights of the current pixel relative to the vertices of the original API primitive prior to clipping (see later re. interactions with clipping).

The 3 values are NOT guaranteed to add up to floating-point 1.0 exactly. If it is desired for the pixel shader to ensure that the weights have this property, it can reconstruct the third coordinate per fragment by subtracting the sum of the other two from 1.0.

Also note, that individual barycentric weights may take on arbitrarily large or arbitrarily small values, and are not constrained to be within [0...1] range necessarily – this may happen for screen-space (non-perspective-correct) barycentric interpolants, screen-space quads, or external triangles.
For triangle primitives, all 3 weights will typically contain non-zero values, but for line primitives the third barycentric weight (e.g. myBaryWeights.z) is guaranteed to be exactly 0.0. With this definition, pixel shaders can remain agnostic to the primitive type being rasterized (i.e. they can evaluate barycentric sums using all 3 barycentric weights for both triangles and lines).

Implementers note: attributes declared as barycentrics will not have space reserved in the pixel shader signature storage.

### Barycentric Coordinate Interpolation Types
For the most part, barycentric coordinates behave just like any other regular attribute, in the sense of allowing either affine (screen-space) or perspective-correct interpolation, and optional centroid adjustment. Interpolation type is specified using existing syntax, e.g.:

    // perspective-correct barycentrics: (default)
    linear float3 BaryLinear : SV_Barycentrics;

    // centroid-adjusted affine (screen-space) barycentrics:
    centroid noperspective float3 BaryAffine : SV_Barycentrics;

    // force shader to run at supersampling rate (one invocation per MSAA sample)
    sample noperspective float3 BaryAffineSample : SV_Barycentrics;

The nointerpolation interpolation modifier is disallowed for barycentric attributes.
As an exception to the current attribute specification rules, it is allowed for the pixel shader to declare **at most two** input attributes with the SV_Barycentrics system semantics, provided that one declaration uses **perspective-correct** interpolation type, and the other uses **noperspective**(affine, screen-space) interpolation type. These two system semantics should have different indices with value either 0 or 1.  For example, the following is a valid attribute syntax:
    
    float4 PSMain(float3 PerspectiveBaryWeights : SV_Barycentrics,
       noperspective float3 NoPerspectiveBaryWeights : SV_Barycentrics1) : SV_Target

This allows the pixel shader to get access to both perspective-correct and affine versions of barycentric coordinates.

### Barycentric Coordinate Ordering
Barycentric coordinates ordering from the perspective of the pixel shader matches vertex ordering of the primitive being rasterized in the original API stream. For example, if a triangle ABC is specified using the following order of its vertices: A, B, C, then the pixel shader will see the barycentric weight associated with the vertex A in the x component of the SV_Barycentric vector, and barycentric weights associated with vertices B and C will be found in y and z components of the vector, correspondingly. Note that this definition also implies that the barycentric weight, associated with the provoking vertex of the current primitive can always be found in the x component of the SV_Barycentrics vector. 
One important exception to the above rule is triangle strips: due to the way triangle strips are specified, the order of barycentric weights will have to be flipped for every other triangle. This is necessary to ensure that the pixel shader always sees consistent winding of vertices for all triangles in the strip. More specifically, given the following triangle strip:

 ![triangle_strip](https://github.com/youngkim93/DirectXShaderCompiler/blob/execution-barycentric/docs/BarycentricTriangleStrip.jpg)

The first two triangles will specify their barycentric weights in the following order (corresponding to x,y,z components of the SV_Barycentric vector):

Triangle 0: [x:0, y:1, z:2], 0 is the provoking vertex

Triangle 1: [x:1, y:3, z:2], 1 is the provoking vertex

Note how barycentric weights corresponding to vertices 2 and 3 of the second triangle have been flipped relative to their API specification order.

## Per-Vertex Attributes
For pixel shaders to perform attribute interpolation using the system-generated barycentric weights, the attributes values at the vertices must be provided.
These can be accessed by declaring the attribute as nointerpolation, and using new intrinsics to read the values at the 3 vertices.
Attributes declared as nointerpolation do not participate in the clipping and interpolation setup stages, but rather their per-vertex values are provided as is to the pixel shader. Implementations are required to preserve all bits in the binary representation of such attributes through the VS/DS/GS-to-PS interface – so that applications can treat them essentially as sets of 32-bit bitfields with application-specific interpretation.
To use the above barycentric weights to interpolate attributes, the values of the attributes at the vertices are provided by the following routine:

    <attributeType> GetAttributeAtVertex( nointerpolation <attributeType> attribute,
                                                                      uint VertexID );

Where **VertexID** is in the range 0..2, and **attributeType** is an attribute declared with **nointerpolation**.

This routine can be used in the following way:

    // enum available in HLSL 2017
    enum VertexID { 
        FIRST = 0,
        SECOND = 1,
        THIRD = 2
    };

    // Linear perspective-correct interpolation of the COLOR attribute
    float3 main( float3 vBaryWeights : SV_Barycentrics,
		 nointerpolation float3 Color : COLOR ) : SV_Target
    {
        float3 vColor;
        . . .

        float3 vColor0 = GetAttributeAtVertex( Color, VertexID::FIRST );  // color at provoking vertex
        float3 vColor1 = GetAttributeAtVertex( Color, VertexID::SECOND ); // color at 2nd vertex
        float3 vColor2 = GetAttributeAtVertex( Color, VertexID::THIRD );  // color at last vertex

        vColor = vBaryWeights.x*vColor0 + vBaryWeights.y*vColor1 + vBaryWeights.z*vColor2;

        return vColor;
    }

The triangle indices specified follow the order of the original triangle vertices as specified in the API stream, (taking the behavior of strips into account). This also means that the order of barycentric weights and the order of per-vertex attributes always match, so that the pixel shader can always assume that the per-vertex attribute at VertexID= 0 corresponds to the barycentric weight placed in the x component of the SV_Barycentrics vector, etc. This also means that the value of a per vertex attribute at VertexID 0 always corresponds to its value at the “provoking vertex”.

As a consequence, it should be possible for pixel shaders to read attributes directly from a vertex buffer and interpolate them in a manner consistent with the fixed-function interpolator.

As always, it is illegal to combine nointerpolation mode with centroid or sample specifiers on the same parameter.

## Interactions with Clipping
Per-vertex attributes are never clipped, and always passed along unmodified, so that the pixel shader gets access to their values as they are output by the last shader in the geometry pipeline (DS, VS or GS). This is also true for vertices which happen to be outside the frustum, or even behind the W=0 plane in clip-space, regardless of what kind of clipping situation occurs for the current primitive.

## Wireframe mode
When drawing primitives with wireframe rasterizer mode (i.e., D3D12_FILL_MODE_WIREFRAME), barycentric co-ordinates follow the same order as the input vertices. This means that although a barycentric co-ordinate in a wireframe tends to have a zero component and two non-zero ones, the 3rd component isn't necessary the zero.