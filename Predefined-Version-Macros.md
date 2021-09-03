Starting with the March 2020 release, a collection of predefined macros indicating version and compile target information are available from within the shader code. These allow preprocessor conditionals to depend on target shader model version, target shader stage, language version, or compiler release version. Using these macros, a single shader can adapt to multiple versions or invocation parameters accordingly.

# Shader Target Defines

The shader target indicated by the `-T <stage>_<major>_<minor>` option is reflected in the compiler through shader stage and shader model version defines.

## Shader Stage Defines

A series of defines with constant values represent the possible shader stages:
* `__SHADER_STAGE_VERTEX`
* `__SHADER_STAGE_PIXEL`
* `__SHADER_STAGE_GEOMETRY`
* `__SHADER_STAGE_HULL`
* `__SHADER_STAGE_DOMAIN`
* `__SHADER_STAGE_COMPUTE`
* `__SHADER_STAGE_AMPLIFICATION`
* `__SHADER_STAGE_MESH`
* `__SHADER_STAGE_LIBRARY`

The predefined macro, `__SHADER_TARGET_STAGE`, is set at compilation time depending on the option provided to the `-T` flag to one of the above values. To use this value to conditionally include code, `__SHADER_TARGET_STAGE` should be compared with one of the above predefined constants. For example:

```hlsl
#if __SHADER_TARGET_STAGE == __SHADER_STAGE_AMPLIFICATION
  DispatchMesh(8, 8, 1, pld);
#endif
```

## Shader Model Version Defines

The shader model version targeted by the compile is available in the shader code by the `__SHADER_TARGET_MAJOR` and `__SHADER_TARGET_MINOR` defined integers. These contain the <major> and <minor> versions respectively as specified at compile time.

They can be used to enable features where the targeted shader model allows.

```hlsl
#if __SHADER_TARGET_MINOR < 6
RWTexture2D<int64_t> myTexture : register (u0);
#endif
...
#if __SHADER_TARGET_MINOR >= 6
    RWTexture2D<int64_t> myTexture = ResourceDescriptorHeap[0];
#endif


```

Note that these reflect the shader model specified at compile time and not the maximum shader model supported.

# HLSL Language Version Define

The `__HLSL_VERSION` macro contains an integer representing the HLSL language version specified by the `-HV <version>` flag or the default the compiler uses. Current options are:

* 2016 - compatibility version useful for compatibility with FXC
* 2018 - Current default version
* 2021 - incomplete, unreleased version as yet unavailable in main branch

# Compiler Release Version Defines

The `__hlsl_dx_compiler` define has been available for all versions of DXC. It is defined by DXC, but not by the FXC compiler and can be used to differentiate between the two.

Predefined integers representing the four components of the version information of the current compiler.
* __DXC_VERSION_MAJOR
* __DXC_VERSION_MINOR
* __DXC_VERSION_RELEASE
* __DXC_VERSION_COMMITS

As reported by `dxc -?` or `dxc --version` (where available). A release compiler has a form similar to: "1.6.2106.0" is major version of 1, minor of 6, release 2106, and the commits number is 0. Builds from the main branch outside a release will be of the form "1.6.0.2955" where the final number represents the number of commits with a release number of 0. If the shader is to be compiled with non-release compilers, this will have to be accounted for.

This can be useful to work around bugs in a given version:

```hlsl
#if __DXC_VERSION_MAJOR == 1 && (__DXC_VERSION_MINOR < 6 || \
                                 (__DXC_VERSION_MINOR == 6 && __DXC_VERSION_RELEASE <= 2106 && __DXC_VERSION_RELEASE > 0))
  return WARBug12345(Outputs);
#else
  return Outputs;
#endif
```
