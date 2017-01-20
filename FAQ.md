##What is Shader Model 6?
<link to github list of shader versions>
Shader model v6 is the first version of HLSL to be implemented on the Clang/LLVM framework. To make it easy to integrate into existing projects, this new version is designed to be as identical as possible to the previous version (v5.1). All existing shaders should ’just work’, but you get the stability and robustness of a foundation containing 1000s of developer contribution hours.

##What is DXIL?
DXIL (DirectX Intermediate Language) is a version of LLVM IR v3.7 with additions to support the key intrinsics and features of HLSL. The initial posting of this project uses DXIL version 0.7 (beta), but will shortly be updated to DXIL v1.0 for release.

##What are the new features of this version (v6.0):
The primary goal of this first release is parity and consistency with existing shaders, however, some new features have been added:
	1) support for 64-bit integer data types (int64, uint64),
	2)  the wave math intrinsics.

##What is WaveMath?
A new set of intrinsics for use in HLSL that enable operations across lanes in the SIMD processor cores.
These can help the performance of certain algorithms like culling and packing sparse data sets. Documentation is [here](https://github.com/Microsoft/DirectXShaderCompiler/wiki/Wave-Math-Intrinsics)

##What is Experimental mode, and how do I enable experimental features?
Experimental mode lets software developers collaborate with each other and with IHVs on prototyping of new features on GPU drivers. Here is how to use it:

1. Turn on Developer Mode in your OS:
    Settings -> Update&Security -> For Developers -> (*) Developer Mode
2. Enable experimental mode in your app by calling this routine:
    D3D12EnableExperimentalFeatures( D3D12ExperimentalShaderModels );
3. Acquire a driver (or software renderer) that supports experimental mode

##Why use Clang/LLVM?
15 years ago, HLSL shaders were no more than 64 instructions, and typically 12.
So the first version of the HLSL compiler was optimized for very short shaders. Now many codebases involve 1000s of shaders each with 1000s of lines of code. GPU programming is now much closer to CPU programming in terms of scale, so we decided to rebuilding the compiler on a trusted/stable foundation with an excellent full-scale track record. When choosing a new basis for a strategic element like the GPU compiler, we recognized the industry cohesion around Clang and LLVM. By basing the new version on this foundation we enable GPU developers to benefit from, and build upon the large ecosystem of tools, utilities, documentation, core technologies, and expertise of the Clang/LLVM framework.
	Importers, exporters, translators, pretty printer (unparsers), intellisense, debugging, etc.

##What happens to the old compiler (fxc)?
Fxc will continue to be supported (bug fixes, etc) so you can continue to use it.
However, it will not support shader models beyond the current  v5.1, so new features will not be available via that path.
Fxc will also continue to generate the original DXBC byte code, not the LLVM-derived DXIL.

##How does the new compiler integrate into the Windows SDK?
The version of this compiler that is available at that time will be included in each Windows SDK version release.
Newer, or beta versions will be available from this site.

##Can I add new language features to the DirectX Shader Compiler project?
Yes, you can: Language features are those which do not require changes to the underlying hardware or drivers. Examples include true enums, bitfield notation, protected/private, etc. If there is enough demand for those features, we will be happy to accept your pull request and your features can ship in the next version of the official compiler.

##Can I add new hardware-based features to the language?
Yes, you can, but it’s a bit more work: To add new capabilities to the hardware feature set requires changes to the GPU driver underneath, and that involves working closely with the GPU and platform vendors. To facilitate this, the new Experimental Mode in Direct3D has been added to provide a safe way to prototype new features. This lets groups of developers and hardware vendors converge on the optimal design of a feature before making it available to general end users/consumers.
