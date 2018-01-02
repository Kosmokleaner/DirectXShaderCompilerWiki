DirectXShaderCompiler also supports translating HLSL into the [SPIR-V][spirv]
intermediate language. This wiki page details how to build and invoke SPIR-V
CodeGen, and also how to contribute.

For how HLSL features are mapped into SPIR-V, please see the [HLSL to SPIR-V
mapping doc][mapping-doc].

_Note_: SPIR-V CodeGen is still under development.

## Building

Before building, please make sure that you have all the prerequisite tools
listed on the [Building Sources][build-source] page.

Assume that you clone the source code into `<dxc-src-dir>` and generate the
artifacts into `<dxc-bin-dir>`:

```sh
git clone git@github.com:Microsoft/DirectXShaderCompiler.git <dxc-src-dir>
cd <dxc-src-dir>
git submodule update --init

# Run the following if Windows Driver Kit is not installed
python .\utils\hct\hctgettaef.py

.\utils\hct\hctstart.cmd <dxc-src-dir> <dxc-bin-dir> # Set up environment
.\utils\hct\hctbuild.cmd -spirv                      # Configure and build
```

If the build is successful, the generated `dxc.exe` should have SPIR-V CodeGen
built in.

## Using

To translate HLSL source file `<hlsl-src-file>` into SPIR-V binary
`<spirv-bin-file>`:

```sh
\path\to\dxc.exe -spirv <hlsl-src-file> -Fo <spirv-bin-file>
```

Apart from the common command-line options like `-O`, `-Fo`, `-Fh`, there are
a few Vulkan/SPIR-V specific [command-line options][vulkan-cl-options] that
you can use and `dxc.exe -help` explains them.

## Contributing

The SPIR-V CodeGen is mainly contributed and maintained by the @google/shaderc
team. But contributions to the SPIR-V CodeGen are definitely very welcome! :)

In addition to the DirectXShaderCompiler [contributing
guidelines][dxc-contribute], please also make sure to follow the following
guidelines.

### Developing

When configuring, please make sure to enable building SPIR-V tests:

```sh
.\utils\hct\hctbuild.cmd -s -spirvtest # Configure with SPIR-V CodeGen & tests
.\utils\hct\hctbuild.cmd -b            # Build
```

### Pull requests and code review

For each pull request, please make sure

- Tests are written to cover the modifications.
- [The HLSL to SPIR-V mapping doc][mapping-doc] is updated for newly supported
  features.

We use [GoogleTest][googletest] as the unit test and CodeGen test framework.
Appveyor will be used to check regression of all pull requests.

Read more about the SPIR-V CodeGen internals in the following section.

## Internals

Various designs are driven by technical considerations together with the
following guidelines for good citizenship within DirectXShaderCompiler:

- Conduct minimal changes to existing interfaces and libraries
- Perfer less intrusive solutions

### General approach

The general approach is to translate frontend AST directly into SPIR-V binary.
We choose this approach considering that

- Frontend AST is much more higher-level than DXIL. For example, [DXIL
  scalarized vectors][dxil-vector] but SPIR-V has native support.
- DXIL has widely different semantics than Vulkan flavor of SPIR-V. For example,
  [structured control flow is not preserved in DXIL][dxil-control-flow]
  but SPIR-V for Vulkan requires it.
- Frontend AST perserves the information in the source code better.
- Also, the right place to generate error messages is in Clang's semantic
  analysis step, which is when the compiler is still processing the AST.

Therefore, it is easier to go from frontend AST to SPIR-V than from DXIL since
we do not need to rediscover certain information.

#### LLVM optimization passes

Translating frontend AST directly into SPIR-V binary precludes the usage of
existing LLVM optimization passes. This is expected since there are also subtle
semantics differences between SPIR-V and LLVM IR. Certain concepts in SPIR-V
do not have direct corresponding representation in LLVM IR and there are no
existing translation schemes handling the differences. Using vanilla LLVM
optimization passes will likely violate the requirements of SPIR-V and results
in invalid SPIR-V modules.

Instead, optimizations are available in the [SPIRV-Tools][spirv-tools] project.

### Library

On the library side, this means introducing a new `ASTFrontendAction` and a
SPIR-V module builder. The new frontend action will traverse the AST and call
the SPIR-V module builder to construct SPIR-V words. These code should be
placed at `tools/clang/lib/SPIRV` and packed into one library (or multiple
libraries in the future).

Detailed design will be revised to accommodate more and more HLSL features.
At the moment, we have:

```
              EmitSPIRVAction
                   |
                   | creates
                   V
              SPIRVEmitter
                   |
                   | contains
                   |
     +-------------+------------+
     |                          |
     |                          |
     V         references       V
SPIRVContext <------------ ModuleBuilder
                                |
                                | contains
                                V
                            InstBuilder
                                |
                                | depends on
                                V
                           WordConsumer
```

- `SPIRVEmitter`: The derived `ASTConsumer` which acts on various frontend
  AST nodes by calling corresponding `ModuleBuilder` methods to build SPIR-V
  modules gradually.
- `ModuleBuilder`: Exposes API for constructing SPIR-V modules. Internally it
  has structured representation of SPIR-V modules, functions, basic blocks as
  well as various SPIR-V specific structs like entry points, debug names, and
  so on.
- `SPIRVContext`: Responsible for <result-id> allocation and maintaining the
  lifetime of objects allocated to represent types, decorations, and others.
  It is used in conjunction with `ModuleBuilder`.
- `InstBuilder`: The low-level interface for generating SPIR-V words for
  various SPIR-V instructions. All SPIR-V instructions are eventually
  serialized via `InstBuilder`.
- `WordConsumer`: The consumer of generated SPIR-V words.

### Command-line tool

A new option, ``-spirv``, will be added into ``dxc.exe`` for invoking the new
SPIR-V CodeGen action.

### Testing

[GoogleTest][googletest] is used as both the unit test and the SPIR-V
CodeGen test framework.

Unit tests are placed under the `tools/clang/unittests/SPIRV/` directory,
while SPIR-V CodeGen tests are placed under the `tools/clang/test/CodeGenSPIRV/`
directory.

For SPIR-V CodeGen tests, there are two test fixtures: one for checking the
whole disassembly of the generated SPIR-V code, the other is
[FileCheck][filecheck]-like, for partial pattern matching.

- **Whole disassembly check**: These tests are in files with suffix
  `.hlsl2spv`. Each file consists of two parts, HLSL source code input and
  expected SPIR-V disassembly ouput, delimited by `// CHECK-WHOLE-SPIR-V:`.
  The compiler takes in the whole file as input and compile its into SPIR-V
  binary. The test fixture then disasembles the SPIR-V binary and compares the
  disassembly with the expected disassembly listed after
  `// CHECK-WHOLE-SPIR-V`.
- **Partial disassembly match**: These tests are in files with suffix `.hlsl`.
  [Effcee][effcee] is used for the stateful pattern matching. Effcee itself
  depends on a regular expression library, [RE2][re2]. See Effcee for supported
  `CHECK` syntax. They are largely the same as LLVM FileCheck.

### Dependencies

SPIR-V CodeGen functionality will require two external projects:
[SPIRV-Headers][spirv-headers] (for `spirv.hpp11`) and
[SPIRV-Tools][spirv-tools] (for SPIR-V disassembling and optimizations).
These two projects should be checked out under the `external/` directory.

The three projects for testing, GoogleTest, Effcee, and RE2, should also be
checked out under the `external/` directory.

### Build system

SPIR-V CodeGen functionality will structured as an optional feature in
DirectXShaderCompiler. Two new CMake options will be introduced to control the
configuring and building SPIR-V CodeGen:

- `ENABLE_SPIRV_CODEGEN`: If turned on, enables the SPIR-V CodeGen
  functionality. (Default: OFF)
- `SPIRV_BUILD_TESTS`: If turned on, enables building of SPIR-V related tests.
  This option will also implicitly turn on `ENABLE_SPIRV_CODEGEN`.
  (Default: OFF)

For building, `hctbuild` will be extended with two new switches, `-spirv`
and `-spirvtest`, to turn on the above two options, respectively.

For testing, `hcttest spirv` will run all existing tests together with SPIR-V
tests, while `htctest spirv_only` will only trigger SPIR-V tests.


[build-source]: https://github.com/Microsoft/DirectXShaderCompiler/wiki/Building-Sources
[dxil-control-flow]: https://github.com/Microsoft/DirectXShaderCompiler/blob/master/docs/DXIL.rst#control-flow-restrictions
[dxil-vector]: https://github.com/Microsoft/DirectXShaderCompiler/blob/master/docs/DXIL.rst#vectors
[dxc-contribute]: https://github.com/Microsoft/DirectXShaderCompiler/blob/master/CONTRIBUTING.md
[effcee]: https://github.com/google/effcee
[filecheck]: https://llvm.org/docs/CommandGuide/FileCheck.html
[googletest]: https://github.com/google/googletest
[google-fork]: https://github.com/google/DirectXShaderCompiler
[mapping-doc]: https://github.com/Microsoft/DirectXShaderCompiler/blob/master/docs/SPIR-V.rst
[re2]: https://github.com/google/re2
[spirv]: https://www.khronos.org/registry/spir-v
[spirv-headers]: https://github.com/KhronosGroup/SPIRV-Headers
[spirv-tools]: https://github.com/KhronosGroup/SPIRV-Tools
[vulkan-cl-options]: https://github.com/Microsoft/DirectXShaderCompiler/blob/master/docs/SPIR-V.rst#vulkan-command-line-options
