## What is DXIL?
DXIL (DirectX Intermediate Language) is a version of LLVM IR v3.7 with additions to support the key intrinsics and features of HLSL. The initial posting of this project uses DXIL version 0.7 (beta), but will shortly be updated to DXIL v1.0 for release.

## What is Experimental mode, and how do I enable experimental features?
Experimental mode is a new feature of Direct3D in Windows 10. It lets software developers collaborate with each other and with IHVs on prototyping of new features on GPU drivers. Here is how to access it:

1. Turn on Developer Mode in your OS:  
  Settings -> Update&Security -> For Developers -> (*) Developer Mode
2. Enable an experimental mode feature in your app by calling this routine before calling `CreateDevice()`.  
  `D3D12EnableExperimentalFeatures( D3D12ExperimentalShaderModels );`
3. Acquire a driver (or software renderer) that supports experimental mode

## Why use Clang/LLVM?
15 years ago, HLSL shaders were no more than 64 instructions, and typically 12.

So the first version of the HLSL compiler was optimized for very short shaders. Now many codebases involve 1000s of shaders each with 1000s of lines of code. GPU programming is now much closer to CPU programming in terms of scale, so we decided to rebuild the compiler on a trusted/stable foundation with an excellent full-scale track record. When choosing a new basis for a strategic element like the GPU compiler, we recognized the industry cohesion around Clang and LLVM. By basing the new version on this foundation we enable GPU developers to benefit from, and build upon the large ecosystem of tools, utilities, documentation, core technologies, and expertise of the Clang/LLVM framework, including importers, exporters, translators, pretty printers (unparsers), code analysis, debugging, etc.

## What happens to the old compiler (fxc)?
fxc is no longer under active development.  We are only fixing high priority bugs in fxc that we cannot offer a workaround for.  Also note that fxc only ships as a component of the Windows SDK and we are unable to ship a fix outside of the Windows SDK release process.  If you have a high priority fxc bug that you would like us to investigate, please mail askhlsl@microsoft.com.

fxc will not support shader models beyond the current v5.1, so new features will not be available via that path.

Fxc will also continue to generate the original DXBC byte code, not the LLVM-derived DXIL.

Developers are strongly encouraged to port their shaders to DXC wherever possible: [[Porting-shaders-from-FXC-to-DXC]]

## How does the new compiler integrate into the Windows SDK?
The version of this compiler that is available at that time will be included in each Windows SDK version release.

Newer, or beta versions will be available from this site.

## Can I add new language features to the DirectX Shader Compiler project?
Yes, you can: Language features are those which do not require changes to the underlying hardware or drivers. Examples include true enums, bitfield notation, protected/private, etc. If there is enough demand for those features, we will be happy to accept your pull request and your features can ship in the next version of the official compiler.

## Can I add new hardware-based features to the language?
Yes, you can, but it's a bit more work: To add new capabilities to the hardware feature set requires changes to the GPU driver underneath, and that involves working closely with the GPU and platform vendors. To facilitate this, the new Experimental Mode in Direct3D has been added to provide a safe way to prototype new features. This lets groups of developers and hardware vendors converge on the optimal design of a feature before making it part of the official specification and available to general end users/consumers.

