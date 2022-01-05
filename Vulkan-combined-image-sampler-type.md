### Design
For the cross language compatibility with GLSL and the advantage of performance on some hardware, we decided to support Vulkan [combined image samplers](https://www.khronos.org/registry/vulkan/specs/1.2/html/chap14.html#descriptorsets-combinedimagesampler).

For a better design of this new DXC/SPIR-V feature, we keep the following things in mind:

* At all possible do not introduce a forceful case for #ifdef - this needs to be left up to the user.
* At all possible do not introduce new types and functions unless absolutely necessary.
* Register binding logistics is the responsibility of the application, not the shader compiler. The shader compiler will do whatever the user tells it to.

### Definition of Texture and SamplerState with `[[vk::combinedImageSampler]]`
You have to define a Texture (e.g., `Texture2D`, `Texture1DArray`, `TextureCube`) and `SamplerState` with the same descriptor set and binding numbers to use the combined image sampler (or sampled image) type.

The command line option `-fvk-t-shift` can be used to apply shifts to combined texture and sampler resource bindings. When `[[vk::combinedImageSampler]]` is applied to a resource, the `-fvk-t-shift` value will be used to shift its bindings and any `-fvk-s-shift` value will be ignored.

See the following examples. Note that the following shader examples will work fine for both Vulkan and DirectX.
```
// Create a combined image/sampler at binding=2
// Register for myTexture and mySampler are t2, s2 in DX.
[[vk::combinedImageSampler]]
Texture2D<float4> myTexture : register(t2);
[[vk::combinedImageSampler]]
SamplerState mySampler : register(s2);

// Create a combined image/sampler at binding=2
// t# and s# are auto assigned for DX. 
[[vk::combinedImageSampler]][[vk::binding(2)]]
Texture2DArray myTexture;
[[vk::combinedImageSampler]][[vk::binding(2)]]
SamplerState mySampler;

// Create a combined image/sampler at binding=2
// Both registers for myTexture and mySampler are ignored in Vulkan.
// Register for myTexture and mySampler are t4, s6 in DX.
[[vk::combinedImageSampler]][[vk::binding(2)]]
TextureCube myTexture : register(t4);
[[vk::combinedImageSampler]][[vk::binding(2)]]
SamplerState mySampler : register(s6);

// Create a combined image/sampler at binding=2
// Register for myTexture is ignored in Vulkan.
// Register for myTexture is t4 in DX. 
// Register for mySampler is autoassigned in DX;
[[vk::combinedImageSampler]][[vk::binding(2)]]
Texture2D<float4> myTexture : register(t4);
[[vk::combinedImageSampler]][[vk::binding(2)]]
SamplerState mySampler;

// Create a combined image/sampler at binding=2
// Register for myTexture is ignored in Vulkan.
// Register for myTexture is autoassigned in DX.
// Register for mySampler is s4 in DX.
[[vk::combinedImageSampler(2)]]
Texture2D<float4> myTexture;
[[vk::combinedImageSampler(2)]]
SamplerState mySampler : register(s4);

// Invalid since myTexture and mySampler has no connection.
[[vk::combinedImageSampler]] Texture2D    myTexture;
[[vk::combinedImageSampler]] SamplerState mySampler : register(s4);

// Invalid since myTexture and mySampler has no connection.
[[vk::combinedImageSampler]] Texture2D    myTexture : register(t4);
[[vk::combinedImageSampler]] SamplerState mySampler;
```

### Usage of Texture and SamplerState defined with `[[vk::combinedImageSampler]]`
See the following example:
```
// Create a combined image/sampler at binding=1
// Register for textureArray and samplerArray are t1, s1 in DX.
[[vk::combinedImageSampler]]
Texture2DArray textureArray : register(t1);
[[vk::combinedImageSampler]]
SamplerState samplerArray : register(s1);

struct VSOutput
{
[[vk::location(0)]] float3 Normal : NORMAL0;
[[vk::location(1)]] float3 Color : COLOR0;
[[vk::location(2)]] float3 UV : TEXCOORD0;
[[vk::location(3)]] float3 ViewVec : TEXCOORD1;
[[vk::location(4)]] float3 LightVec : TEXCOORD2;
};

float4 main(VSOutput input) : SV_TARGET
{
	float4 color = textureArray.Sample(samplerArray, input.UV) *
                    float4(input.Color, 1.0);
	float3 N = normalize(input.Normal);
	float3 L = normalize(input.LightVec);
	float3 V = normalize(input.ViewVec);
	float3 R = reflect(-L, N);
	float3 diffuse = max(dot(N, L), 0.1) * input.Color;
	float3 specular = (dot(N,L) > 0.0) ?
         pow(max(dot(R, V), 0.0), 16.0) * float3(0.75, 0.75, 0.75) *
             color.r :
         float3(0.0, 0.0, 0.0);
	return float4(diffuse * color.rgb + specular, 1.0);
}
```

You can test the above code with [this example](https://github.com/SaschaWillems/Vulkan/tree/master/examples/instancing) by replacing [instancing.frag](https://github.com/SaschaWillems/Vulkan/blob/master/data/shaders/hlsl/instancing/instancing.frag) with the above HLSL code. You have to compile it and replace [instancing.frag.spv](https://github.com/SaschaWillems/Vulkan/blob/master/data/shaders/hlsl/instancing/instancing.frag.spv) with it.

We can use `textureArray` and `samplerArray` in following ways:
* Use them together e.g., `textureArray.Sample(samplerArray, input.UV)`
  * The generated SPIR-V will load the combined image sampler and use it with [image instructions](https://www.khronos.org/registry/SPIR-V/specs/unified1/SPIRV.html#_a_id_image_a_image_instructions).
* Use `textureArray` with samplers other than `samplerArray`
  * The generated SPIR-V will load the combined image sampler, extract the image from the combined image sampler using [OpImage](https://www.khronos.org/registry/SPIR-V/specs/unified1/SPIRV.html#OpImage) and combine it with other samplers using [OpSampledImage](https://www.khronos.org/registry/SPIR-V/specs/unified1/SPIRV.html#OpSampledImage) to use [image instructions](https://www.khronos.org/registry/SPIR-V/specs/unified1/SPIRV.html#_a_id_image_a_image_instructions).

Note that we cannot use `samplerArray` with textures other than `textureArray` because there is no SPIR-V instruction to extract the sampler from a combined image sampler.

Another example is [here](https://github.com/jaebaek/Vulkan/blob/vk_combined_image_sampler/data/shaders/hlsl/deferredshadows/deferred.frag). It is a fragment shader for [SaschaWillems's deferredshadows Vulkan example](https://github.com/jaebaek/Vulkan/tree/vk_combined_image_sampler/examples/deferredshadows). You can
1. clone [this branch](https://github.com/jaebaek/Vulkan/tree/vk_combined_image_sampler)
2. [build](https://github.com/jaebaek/Vulkan/blob/vk_combined_image_sampler/BUILD.md#-linux) it
3. test the deferredshadows example with `--shaders hlsl` option that enables the HLSL fragment shader.