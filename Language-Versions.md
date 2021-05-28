Just like DirectX Shader Compiler supports multiple [[Shader Models]] that evolve over time to reflect hardware capabilities, so does it support multiple language versions. Language versions are typically named after the year of their release, and will occasionally be required to address features of specific shader models (for example, new primitive data types).

Additions of new intrinsics are typically considered shader model revisions, unless they also affect the type system.

## HLSL 2015

This is the version of the HLSL language supported by fxc. It is only partially supported by the DirectX Shader Compiler, mostly with the goal of enabling basic tooling scenarios that require an abstract syntax tree.

## HLSL 2016

This is the first version of the HLSL language that the DirectX Shader Compiler can compile to DXIL. It adds support for int64_t and uint64_t and their vector and matrix forms. It also removes some features from HLSL 2015, see [[Breaking Changes from FXC]].

## HLSL 2017

This version adds enum and enum class declarations.

## HLSL 2018

This version adds width-specific types for floats and ints, for 16, 32 and 64 bit widths.

## Additional Changes

Some features have been added that aren't exactly considered language changes or tied to an HLSL version, and also not tied to a specific shader model.  One such feature is [[ByteAddressBuffer Load Store Additions]].
