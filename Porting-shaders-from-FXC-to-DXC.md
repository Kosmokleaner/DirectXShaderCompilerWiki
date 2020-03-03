# Porting shaders from FXC to DXC

## Introduction

This topic should be used as a reference point when porting your existing high-level shader language (HLSL) shaders over from D3DCompiler (FXC) to DXCompiler (DXC). Specifically, this topic provides details about the following: 

1. Enable some of the old FXC compilation behaviors that are disabled by default on DXC. This is supported on DXC through the following compiler flags:

    - `-Gec`: Enable Direct3D 9 backward compatibility (for example, enable support for `COLOR` semantic that's supported only until Direct3D 9).

    - `-HV`: Enable FXC backward compatibility by setting the language version to 2016 (that is, `-HV 2016`). This enables a certain legacy language semantic that's available on FXC.

    - `-flegacy-macro-expansion`: Expand the operands before performing token-pasting operation (FXC behavior).

    - `-flegacy-resource-reservation`: Reserve unused explicit register assignments for compatibility with shader model 5.0 and earlier.

2. Provide information about the old deprecated HLSL syntaxes that were supported on FXC but are no longer supported on DXC.

## Compiler .dll interface changes
We recommend using the new `IDxcCompiler3::Compile` function for compiling DXC shaders.  For more information about helper routines that will make porting existing code to the new interface easier, along with sample code, see [Using DXC](dxc-usage.md).

## Backward compatibility with FXC behaviors

### Multiplication-only pattern for the intrinsic function [pow](https://docs.microsoft.com/windows/desktop/direct3dhlsl/dx-graphics-hlsl-pow)

By default, DXC always uses *log-mul-exp pattern* (explained as follows) to implement the **pow** function. In contrast, FXC also uses the *mul-only* pattern instead to implement the **pow** function in some scenarios. For example, for smaller constant values of *y* (such as *pow(x, 3)*), FXC uses the *mul-only* pattern. This could lead to the **pow** function producing different results when compiled by using FXC and DXC in some cases. For example, for a runtime value of *x=0*, *pow(x,3)* evaluates to *NaN* and *0* on DXC and FXC, respectively. To achieve the FXC behavior with respect to the **pow** function on DXC, set the language version to 2016 (by using the flag `-HV 2016`) during compilation.

#### Example of pow implementation by using the log-mul-exp pattern

```C++
pow(x,y) = e^(log(x) * y)
```

#### Example of pow implementation by using the mul-only pattern

```C++
pow(x,3) = (x * x) * x 
```

### Out-of-bound array accesses
By default, DXC errors out when finding constant index references to an array, which is out of bound. For example, the following HLSL shader fails to compile with an error message: `error: array index 3 is out of bounds`. You could turn this error into a warning by setting the language version to 2016 (that is, by using the `-HV 2016` flag) only when the out-of-bound access happens in the unused or dead code.

```C++
void main()
{
  uint x[2];
  x[3]=1.f; // Out-of-bound access.
} 
```

### Warn when unroll attribute can't be honored
When the **unroll** attribute is applied on a loop and if the loop-trip&ndash;count can't be ascertained at compile-time or if it's above a certain threshold, then by default, DXC fails to compile shaders such as the following. To turn this failure into a warning, set the language version to 2016. When the following shader is compiled on DXC with the `-HV 2016` flag, it compiles successfully without unrolling the loop and generates a warning message: `warning: Could not unroll the loop.`

```C++
uint main(uint x : IN) : OUT
{
   uint r = 0;
   [unroll]
   for(uint i=0; i<x; i++)
    r+=i;
   return r;
}
```

### COLOR semantic

The semantic COLOR isn't supported on DXC by default. When writing new HLSL shaders targeting DXC, you should instead use [SV_Target](https://docs.microsoft.com/windows/desktop/direct3dhlsl/dx-graphics-hlsl-semantics#migration-from-direct3d-9-to-direct3d-10-and-later). However, DXC also provides an option to compile shaders referencing the COLOR semantic by passing it the `-Gec` flag. For example, the following HLSL code should compile successfully on DXC when the `-Gec` flag is used.

```C++
float main() : COLOR
{
    return 1.f;
}
```

### Length property on constant arrays

The `Length` property on constant arrays is supported on DXC under the `-Gec` flag as shown in the following example.

```C++
uint main() : OUT
{
    int x[5];
    return x.Length;
}
```

### Writing to globals
Global variables are treated as constants in HLSL, but just like FXC, DXC also enables support for writing to globals with the `-Gec` flag as shown in the following example.

```C++
float gvar; // Write enabled with "-Gec"

float4 main(uint a : A) : SV_Target
{
  gvar = a * 2.0f;  
  return (float4) gvar;
} 
```


### Legacy resource reservation by using the `-flegacy-resource-reservation` flag

For shader model 5.0 or earlier, FXC would factor in the register allocation of unused resources when doing register allocation for used ones. For example, when compiling the following shader on FXC with SM5.0 and 5.1, `Tex1[1]` is bound to `t4` and `t0`, respectively.
When using DXC, to achieve the FXC resource allocation behavior with SM5.0 or earlier, use the `-flegacy-resource-reservation` flag.

```C++
Texture2D Tex0[2] : register(t1);
Texture2D Tex1[2];

float4 main() : SV_Target
{
    return Tex1[1].Load((uint3)0);
} 
```

#### Resource binding on FXC with SM5.0

```C++
// Resource bindings:
//
// Name                                 Type  Format         Dim      HLSL Bind  Count
// ------------------------------ ---------- ------- ----------- -------------- ------
// Tex1[1]                           texture  float4          2d             t4      1
```

#### Resource binding on FXC with SM5.1
```C++
// Resource bindings:
//
// Name                                 Type  Format         Dim      ID      HLSL Bind  Count
// ------------------------------ ---------- ------- ----------- ------- -------------- ------
// Tex1                              texture  float4          2d      T0             t0      2
```

#### Resource binding on DXC with SM6.0 without the `-flegacy-resource-reservation` flag

```C++
// Resource bindings:
//
// Name                                 Type  Format         Dim      ID      HLSL Bind  Count
// ------------------------------ ---------- ------- ----------- ------- -------------- ------
// Tex1                              texture     f32          2d      T0             t0     2
```

#### Resource binding on DXC with SM6.0 with the `-flegacy-resource-reservation` flag

```C++
// Resource bindings:
//
// Name                                 Type  Format         Dim      ID      HLSL Bind  Count
// ------------------------------ ---------- ------- ----------- ------- -------------- ------
// Tex1                              texture     f32          2d      T0             t3     2
```

### Legacy macro expansion by using the `-flegacy-macro-expansion` flag
The following HLSL shader compiles successfully on FXC, but errors out on DXC as `b##slot` is replaced with `bSLOT_VAL` instead of `b3`, thus causing compilation failure. To achieve the FXC token replacement behavior on DXC, compile with the additional flag "`-flegacy-macro-expansion`".

```C++
#define SLOT_VAL 3
#define CBUFFER_ALIGNED( name, slot ) cbuffer name : register( b##slot )
 
CBUFFER_ALIGNED( MyCBuffer, SLOT_VAL )
{
       float4        val;
};
```

You can avoid the usage of the -flegacy-macro-expansion flag in this case by following standard preprocessor rules.

```C++
#define SLOT_VAL 3
#define PASTE1(a, b) a##b
#define PASTE(a, b) PASTE1(a, b)
#define CBUFFER_ALIGNED( name, slot ) cbuffer name : register( PASTE(b, slot) )
CBUFFER_ALIGNED( MyCBuffer, SLOT_VAL )
{
       float4        val;
};
```


-flegacy-macro-expansion doesn't help for cases where the tokens are not macro arguments, though fxc still expands these.  The following will result in an error even if -flegacy-macro-expansion is used.

```C++
#define C 4
#define A4 float4
#define TYPE A##C

#define FUNC(ty) ty main() : OUT { return 0; }
FUNC(TYPE)
```

 The following change will bring the HLSL into compatibility with standard preprocessor rules, no longer requiring the -flegacy-macro-expansion option.

```C++
#define C 4
#define A4 float4
#define PASTE1(a, b) a##b
#define PASTE(a, b) PASTE1(a, b)
#define TYPE PASTE(A, C)

#define FUNC(ty) ty main() : OUT { return 0; }
FUNC(TYPE)
```


## Unsupported FXC behaviors

### Dimension-specific texture sample functions no longer supported
Support for [tex1D](https://docs.microsoft.com/windows/desktop/direct3dhlsl/dx-graphics-hlsl-tex1d), [tex2D](https://docs.microsoft.com/windows/desktop/direct3dhlsl/dx-graphics-hlsl-tex2d), and similar functions for sampling are now deprecated. To learn how to port to supported sampling functions, see the following example.


#### Example using the old syntax for sampling, which is no longer supported

```C++
sampler FontTextureSampler : register(s0);
struct VS_OUT
{
   float2 TexCoord0 : TEXCOORD0;
};

float4 PSMain_Unsupported( VS_OUT In ) : SV_Target
{
    return tex2D( FontTextureSampler, In.TexCoord0 ).zyxw; // tex2D is unsupported.
}
```
#### Example showing supported syntax for sampling

```C++
SamplerState FontTextureSampler : register(s0);
Texture2D FontTexture2D : register( t0 ); // Step 1. Declare texture.

struct VS_OUT
{
   float2 TexCoord0 : TEXCOORD0;
};

float4 PSMain( VS_OUT In ) : SV_Target
{
    // Step 2. Use "Sample" method of the texture declared in the previous step.
    return FontTexture2D.Sample( FontTextureSampler, In.TexCoord0 ).zyxw;
}
```

### Semantic overlap not supported
DXC disallows the use of duplicate semantics. For example, the following shader will fail compilation on DXC.

```C++
uint main(uint a : SV_VertexID, uint b : SV_VertexID) : OUT
{
    return a + b;
}
```

### "ConstantBuffer" as constant buffer name not supported

Starting with SM5.1, HLSL supports [indexable constant buffers](https://docs.microsoft.com/windows/desktop/direct3d12/resource-binding-in-hlsl#constant-buffers) by using the _ConstantBuffer_ template construct. Therefore, _ConstantBuffer_ is a reserved keyword
and DXC disallows using it as a buffer name.


```C++
cbuffer ConstantBuffer : register(b0) // ConstantBuffer as buffer name not allowed.
{
 float2 field1;
 float field2;
}
```
