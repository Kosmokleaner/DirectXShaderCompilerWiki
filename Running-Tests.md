To run tests, open the HLSL Console and run this command after a successful build.

    hcttest

Some tests will run shaders and verify their behavior. These tests also involve a driver that can run these execute these shaders. Currently all these tests are grouped under the ExecutionTest class in the `tools/clang/unittests/HLSL` library. See [[this section|Running-Shaders]] to learn how set up these up.

The tests for the DirectX Shader Compiler project are written to be mostly run within a single process, controlled by the Microsoft [Test Authoring And Execution Framework](https://msdn.microsoft.com/en-us/windows/hardware/drivers/taef/index). This framework is used in other test environments and is included in the Windows Driver Kit, and includes a number of features that make it well-suited to scale from short local test runs to broader distributed runs. You need not install the Windows Driver Kit to get it; the `utils\hct\hctgettaef` script will make it available under the externals directory.

If you built using ninja, pass the `-ninja` flag to `hcttest`.

If you want to run file check on single shader, pass `file-check` with filename like this:

     `hcttest file-check "..\CodeGenHLSL\shader-compat-suite\lib_arg_flatten\lib_arg_flatten.hlsl"`
    
If you want to run file check on a directory, pass `compat-suite` with path like this:

     `hcttest compat-suite "..\CodeGenHLSL\shader-compat-suite\lib_arg_flatten"`

By default, hcttest will run file check on tools\clang\test\CodeGenHLSL\shader-compat-suite. Any files added to shader-compat-suite are run automatically. Feel free to add new tests to this directory.
