The HLSL community has [proposed](https://discourse.llvm.org/t/rfc-adding-hlsl-and-directx-support-to-clang-llvm/60783) adding HLSL, DXIL, and SPIR-V support to LLVM's upstream repository.

This effort is pending acceptance by LLVM project maintainers, but there are a few important notes to share with HLSL contributors and users.

# TL;DR

* We are planning to contribute HLSL support to LLVM's main repository
* This will be a feature-by-feature porting process, that will require significant reimplementation
* We will continue supporting the existing DXC source with new features for at least 12 months
* We will continue supporting the existing DXC source with bug fixes for at least 24 months
* We intend to make a drop-in replacement for DXC based on modern LLVM
* We will be bringing all features of DXC into modern LLVM (including SPIR-V)
* The DXIL generated by modern LLVM will be unchanged for drivers and fully backwards compatible
* This will be a public effort, and we will welcome any and all support

# Motivation

The existing HLSL compiler, the DirectX Shader Compiler (DXC), is a fork of LLVM/Clang 3.7. In recent years we've added many new features to HLSL leveraging Clang's C++ support. In expanding the HLSL language we are limited by both the older C++ language support in our fork of Clang, and the sometimes destructive way that HLSL support was added to Clang. We believe that moving HLSL support onto a modern Clang will enable us to bring more modern C++ language features into HLSL, and allow us to take advantage of tooling improvements in modern Clang.

# Contributing to LLVM

In moving to a modern LLVM we also want to stay up to date with changes made by the LLVM community. We could do this in our own fork with merge automation, however we believe it is preferable to contribute HLSL support into the LLVM project so that we can avoid needing to manage merge conflicts and breakages from other open source contributors.

We also believe that in merging our open source community with LLVM's we will gain supporters and gather contributions from a much wider community.

# Process

Our guiding principle is to be good stewards of our open source software and community. The Microsoft team is going to take ownership of getting the LLVM project up to feature parity. We are not expecting contributors to DXC to also make changes to LLVM, although we will welcome any and all support in this effort. DXC will remain the official HLSL compiler until LLVM is able to function as a drop-in replacement.

We plan to drive this as an open source initiative. Once we begin implementation, we'll hold regular meetings open to the public to provide status updates and coordinate efforts.

There will be significant architectural differences between DXC and the LLVM HLSL implementation. To simplify porting code between the two codebases we will do some work in DXC to make DXC more modern LLVM-like.

Some of the architectural differences we have planned are:
* Abstracting DXIL generation into an LLVM target
* Utilization of common LLVM patterns like target triples instead of HLSLModule
* HLSL Semantics as Attributes instead of custom Decls
* HLSL.h implicit header to replace some (or all) AST hard coding

# High-level Milestones

Our plan is to start with compute shader support to get DXIL generation, and the basic compiler and language infrastructure in place. After compute shaders are in place we'll begin working on rendering with Pixel and Vertex shaders. We are going to start with small hand-crafted shaders as tests, and work up to using the DirectX sample applications, and eventually our internal runtime test suites. Our high level outline is:

* Add initial DirectX target in LLVM supporting DXIL generation
* Add HLSL language mode to clang frontend
* Basic compute shader support for SM 6.0 (the first pipeline mode will require a lot of foundational work)
  * First shader (Compute passthrough)
    * HLSL Language infrastructure
    * Builtins, shader models, and basic driver
      * UAV support
      * DXIL generation
      * HLSL.h implicit header
  * Second shader (complex compute)
    * HLSL Semantics
    * HLSL Object support (structs, member functions, this)
* Simple Graphics support
  * First shader (triangle)
    * Rendering semantics
    * Texture samples
  * DirectX Sample Programs
    * Behavior variations from C (logical operators, bit shifts, overload resolution, this)
    * Start building out a runtime test suite
* Mesh, Amplification, Domain and Hull

# Resourcing

From Microsoft's side, Chris ([@llvm-beanz](https://github.com/llvm-beanz)) be taking the lead on the engineering effort with Xiang Li ([@python3kgae](https://github.com/python3kgae)) working almost full time on this effort. We also are planning to have everyone on the team contributing, and as Clang HLSL support nears feature complete we will begin shifting our team resources away from DXC.