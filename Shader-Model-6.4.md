## Packed Dot Product Intrinsics
The new shader model adds the following 3 intrinsics to accelerate dot products on reduced precision data:


### Unsigned Integer Dot-Product of 4 Elements and Accumulate:

`uint32 dot4add_u8packed( uint32 a, uint32 b, uint32 acc );		// ubyte4 a, b;`

 A 4-dimensional unsigned integer dot-product with add. Multiplies together each corresponding pair of unsigned 8-bit int bytes in the two input DWORDs, and sums the results into the 32-bit unsigned integer accumulator. This instruction operates within a single 32-bit wide SIMD lane. The inputs are also assumed to be 32-bit quantities. 

### Signed Integer Dot-Product of 4 Elements and Accumulate:

`int32  dot4add_i8packed( uint32 a, uint32 b,  int32 acc );          	// signed byte4 a, b;`

 A 4-dimensional signed integer dot-product with add. Multiplies together each corresponding pair of signed 8-bit int bytes in the two input DWORDs, and sums the results into the 32-bit signed integer accumulator. This instruction operates within a single 32-bit wide SIMD lane. The inputs are also assumed to be 32-bit quantities.

### Single Precision Floating Point 2-Element Dot-Product and Accumulate:

`float dot2add( half2 a, half2 b, float acc );`

 A 2-dimensional floating point dot product of half2 vectors with add. Multiplies the elements of the two half-precision float input vectors together and sums the results into the 32-bit float accumulator. These instructions operate within a single 32-bit wide SIMD lane. The inputs are 16-bit quantities packed into the same lane. This is not considered a ‘fused’ operation, and so need not emit INF due to fp16 overflow unless the `precise` declaration is used.

This covered under the low-precision feature bit (indicating native half and short support).

## Support for Variable Rate Shading
Variable rate shading is supported as of this shader model. Only one token was added to HLSL. 

`SV_ShadingRate`

Indicates the number of samples per pixel during operation of the current pixel shader thread.

## Library Sub-objects

Check the developer documentation on DirectX raytracing [here](https://docs.microsoft.com/en-us/windows/desktop/direct3d12/direct3d-12-raytracing)
