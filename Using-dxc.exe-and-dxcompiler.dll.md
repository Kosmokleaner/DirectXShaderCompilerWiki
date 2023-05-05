# Using the compiler
DXC supports a command-line interface (dxc.exe) and a .dll interface (dxcompiler.dll, dxcompiler.lib, dxcapi.h, and d3d12shader.h). 

For Windows, the latest compiler binaries can be downloaded from GitHub. We recommend always updating to the [latest release](https://github.com/microsoft/DirectXShaderCompiler/releases).  

Windows
```
dxc_<version>\dxc.exe
dxc_<version>\dxcompiler.dll
dxc_<version>\dxcapi.h
dxc_<version>\d3d12shader.h
dxc_<version>\dxcompiler.lib
```

## DXC command-line interface
A complete list of command-line flags is obtained by running `dxc.exe -?`. If you're familiar with FXC, many of the command-line arguments are the same. Following are some common flags.

```
-D <value>              Define macro.
-E <value>              Entry point name.
-Fd <file>              Write debug information to the specified file or directory; trail \ to autogenerate.
-Fo <file>              Output object file.
-Gec                    Enable backward compatibility mode.
-HV <value>             HLSL version (2016, 2017, 2018, 2021). Default is 2018.
-Od                     Disable optimizations.
-T <profile>            Set target profile.
    <profile>: ps_6_0, ps_6_1, ps_6_2, ps_6_3, ps_6_4, ps_6_5, ps_6_6,
                vs_6_0, vs_6_1, vs_6_2, vs_6_3, vs_6_4, vs_6_5, vs_6_6,
                cs_6_0, cs_6_1, cs_6_2, cs_6_3, cs_6_4, cs_6_5, cs_6_6,
                gs_6_0, gs_6_1, gs_6_2, gs_6_3, gs_6_4, gs_6_5, gs_6_6,
                ds_6_0, ds_6_1, ds_6_2, ds_6_3, ds_6_4, ds_6_5, ds_6_6,
                hs_6_0, hs_6_1, hs_6_2, hs_6_3, hs_6_4, hs_6_5, hs_6_6,
                lib_6_3, lib_6_4, lib_6_5, lib_6_6,
-Vd                     Disable validation.
-Zi                     Enable debug information.
-Zpc                    Pack matrices in column-major order.
-Zpr                    Pack matrices in row-major order.
  ```

>**NOTE:** DXC has been updated to no longer embed .pdb (program database/debug information) by default in the shader. By default, .pdbs are external.

`dxc.exe --version` will also report which variant of the compiler it is. 

Following is an example of using the command-line interface.

```
dxc -E main -T ps_6_0 -Fo myshader.bin -Zi -Fd myshader.pdb -D MYDEFINE=1 myshader.hlsl
```

In this example, myshader.hlsl is compiled to a shader model 6.0 pixel shader. The compiled shader is written to myshader.bin as specified by the `-Fo` flag. Debug information is enabled with the `-Zi` flag, and the .pdb is written out to myshader.pdb with the `-Fd` flag. If you would prefer that the compiler automatically generate a .pdb name, you can supply `.\` as the `-Fd` argument. Again, the debug information in the PDB isn't embedded in the shader.

If the same macro has multiple definitions, such as ```-D MYDEFINE=1 -D MYDEFINE=2```, only the last definition will be used.

## Shader hash

The hash for the shader can be saved to file by using the `-Fsh` command-line flag. Through the .dll interface, it's always available by using `IDxcResult::GetOutput`.  

On Windows, the shader DXIL is hashed to determine the value.  

### Hash format

The blob contents returned using APIs: `IDxcResult::GetOutput(DXC_OUT_SHADER_HASH, ...)` and `IDxcPdbUtils::GetHash(...)`, as well as file contents written using the `-Fsh` option, map to the `DxcShaderHash` structure defined in `dxcapi.h`:

```cpp
// Hash digest type for ShaderHash
typedef struct DxcShaderHash {
  UINT32 Flags; // DXC_HASHFLAG_*
  BYTE HashDigest[16];
} DxcShaderHash;
```

>**NOTE:** The hash blob produced by the `IDxcUtils::GetPDBContents` API does not contain a `DxcShaderHash` structure.  Instead, the hash blob will contain just the raw 16-byte hash digest.

## Reflection

By default, reflection data is stored in the shader. The `-Qstrip_reflect` flag can be used to remove it.  
The flag `-Fre` can now be used to save your reflection data out to a separate file, and the file is saved even if `-Qstrip_reflect` is used. Similarly, `IDxcResult::GetOutput` can be used to get the separate reflection blob, even if `-Qstrip_reflect` is used. `IDxcUtils::CreateReflection` can create a reflection interface from these separate reflection blobs.

## PDBs

When your shader is compiled, the .pdb filename (supplied with the `-Fd` option) and a hash of the shader is stored in the shader. PIX uses this information to find the shader .pdb.  

### Slim PDBs
A new compilation flag, `-Zs` creates a new type of .pdb that is far smaller than traditional .pdbs and decreases compilation time when .pdbs are generated.  These “slim” .pdbs contain
only the source code and compilation arguments used to generate the shader.  PIX will generate the additional debug information required “on demand” when the shader is debugged.
 
Note that full .pdbs (which contain everything a slim .pdb does plus the full debug information used by the debugger) are still supported with the `-Zi` flag.
 
### A note on compiler versioning.
When a .pdb is generated, the version of the compiler is stored in the .pdb so that PIX can know which compiler was use to originally compiler the shader.  PIX will report if the current compiler associated with PIX does not match the one used to generate the shader and the matching compiler can be supplied if desired.
 
### New pdb interface.
A new interface call IDxcPdbUtils has been introduced in dxcapi.h.  This interface allows you to inspect the source, compilation arguments, compiler version, and hash as well as allows you to recompile the shader from the pdb if desired.

## DXCompiler DLL interface

To use the .dll interface, include dxcapi.h and d3d12shader.h and link to the dxcompiler.lib library or load dxcompiler.dll directly.  

>**NOTE:** Manually loading and unloading other .dlls required by the compiler isn't supported. We recommend that you leave the .dll loaded for the life of our program instead of loading and unloading it for each shader compilation.

After the compiler .dll is loaded, you call `DxcCreateInstance` to create the various compiler interfaces.

>**NOTE:** The call to `DxcCreateInstance` is thread-safe, but objects created by `DxcCreateInstance` aren't thread-safe. Compiler objects should be created and then used on the same thread.


### Using the compiler interface

We recommend that you use the latest `IDxcCompiler3` interface to take advantage of the latest functionality, including the ability to specify more arguments. For example, just like on the command line, named blobs, and direct access to the shader hash. If you're porting from `IDxcCompiler2`, you can use `IDxcUtils::BuildArguments` if you prefer an interface that's similar to `IDxcCompiler::Compile` or `IDxcCompiler2::CompileWithDebug`.

While the IDxcCompiler3::Compile interface resembles the command line interface, arguments are passed as individual strings. Also, quotes used to group arguments with spaces into one argument for the command line can be omitted and escape sequences required for special command line characters are not needed.

<table>
<tr> <th>Command Line</th> <th>Incorrect IDxcCompiler3::Compile arg string</th> <th>Correct IDxcCompiler3::Compile arg string</th> </tr>
<tr> <th>-E main</th> <th>L"-E main"</th> <th>L"-E", L"main"</th></tr>
<tr> <th>-Fd "my filename with spaces.pdb"</th> <th>L"-Fd", L"\"my filename with spaces.pdb\""</th> <th>L"-Fd", L"my filename with spaces.pdb" </th></tr>
<tr> <th>this^&that"</th> <th>L"this^&that"</th> <th>L"this&that"</th></tr>
<tr> <th>-D MY_CONSTANT=2.2</th> <th>L"-D MY_CONSTANT=2.2"</th> <th>L"-D", L"MY_CONSTANT=2.2"</th></tr>
<tr> <th>-D "MY_CODE= 2 + 2"</th> <th>L"-D \"MY_CODE= 2 + 2\""</th> <th>L"-D", L"MY_CODE= 2 + 2"</th></tr>
</table>


The following example code uses the interface to compile myshader.hlsl while generating a .pdb. The general flow is to call `IDxcCompiler3::Compile`, and then investigate `IDxcResult` for the error buffer, compiled shader, .pdb, and shader hash.

>**NOTE:** The .pdb name must be sent in to the `Compile` function by using the `-Fd` argument. Just like on the command line, `-Fd .\` can be used to have the compiler autogenerate the PDB name. If the PDB name is autogenerated, you must use this name to save your .pdb file to so that PIX can find it.

In the following example, common error checking has been omitted for brevity. For more detailed error checking, see [[Advanced Error Handling]].

```cpp
#include <atlbase.h>        // Common COM helpers.
#include <dxcapi.h>         // Be sure to link with dxcompiler.lib.
#include <d3d12shader.h>    // Shader reflection.

int main()
{
    // 
    // Create compiler and utils.
    //
    CComPtr<IDxcUtils> pUtils;
    CComPtr<IDxcCompiler3> pCompiler;
    DxcCreateInstance(CLSID_DxcUtils, IID_PPV_ARGS(&pUtils));
    DxcCreateInstance(CLSID_DxcCompiler, IID_PPV_ARGS(&pCompiler));

    //
    // Create default include handler. (You can create your own...)
    //
    CComPtr<IDxcIncludeHandler> pIncludeHandler;
    pUtils->CreateDefaultIncludeHandler(&pIncludeHandler);


    //
    // COMMAND LINE:
    // dxc myshader.hlsl -E main -T ps_6_0 -Zi -D MYDEFINE=1 -Fo myshader.bin -Fd myshader.pdb -Qstrip_reflect
    //
   LPCWSTR pszArgs[] =
    {
        L"myshader.hlsl",            // Optional shader source file name for error reporting
                                     // and for PIX shader source view.  
        L"-E", L"main",              // Entry point.
        L"-T", L"ps_6_0",            // Target.
        L"-Zs",                      // Enable debug information (slim format)
        L"-D", L"MYDEFINE=1",        // A single define.
        L"-Fo", L"myshader.bin",     // Optional. Stored in the pdb. 
        L"-Fd", L"myshader.pdb",     // The file name of the pdb. This must either be supplied
                                     // or the autogenerated file name must be used.
        L"-Qstrip_reflect",          // Strip reflection into a separate blob. 
    };


    //
    // Open source file.  
    //
    CComPtr<IDxcBlobEncoding> pSource = nullptr;
    pUtils->LoadFile(L"myshader.hlsl", nullptr, &pSource);
    DxcBuffer Source;
    Source.Ptr = pSource->GetBufferPointer();
    Source.Size = pSource->GetBufferSize();
    Source.Encoding = DXC_CP_ACP; // Assume BOM says UTF8 or UTF16 or this is ANSI text.


    //
    // Compile it with specified arguments.
    //
    CComPtr<IDxcResult> pResults;
    pCompiler->Compile(
        &Source,                // Source buffer.
        pszArgs,                // Array of pointers to arguments.
        _countof(pszArgs),      // Number of arguments.
        pIncludeHandler,        // User-provided interface to handle #include directives (optional).
        IID_PPV_ARGS(&pResults) // Compiler output status, buffer, and errors.
    );

    //
    // Print errors if present.
    //
    CComPtr<IDxcBlobUtf8> pErrors = nullptr;
    pResults->GetOutput(DXC_OUT_ERRORS, IID_PPV_ARGS(&pErrors), nullptr);
    // Note that d3dcompiler would return null if no errors or warnings are present.
    // IDxcCompiler3::Compile will always return an error buffer, but its length
    // will be zero if there are no warnings or errors.
    if (pErrors != nullptr && pErrors->GetStringLength() != 0)
        wprintf(L"Warnings and Errors:\n%S\n", pErrors->GetStringPointer());

    //
    // Quit if the compilation failed.
    //
    HRESULT hrStatus;
    pResults->GetStatus(&hrStatus);
    if (FAILED(hrStatus))
    {
        wprintf(L"Compilation Failed\n");
        return 1;
    }

    //
    // Save shader binary.
    //
    CComPtr<IDxcBlob> pShader = nullptr;
    CComPtr<IDxcBlobUtf16> pShaderName = nullptr;
    pResults->GetOutput(DXC_OUT_OBJECT, IID_PPV_ARGS(&pShader), &pShaderName);
    if (pShader != nullptr)
    {
        FILE* fp = NULL;

        _wfopen_s(&fp, pShaderName->GetStringPointer(), L"wb");
        fwrite(pShader->GetBufferPointer(), pShader->GetBufferSize(), 1, fp);
        fclose(fp);
    }

    //
    // Save pdb.
    //
    CComPtr<IDxcBlob> pPDB = nullptr;
    CComPtr<IDxcBlobUtf16> pPDBName = nullptr;
    pResults->GetOutput(DXC_OUT_PDB, IID_PPV_ARGS(&pPDB), &pPDBName);
    {
        FILE* fp = NULL;

        // Note that if you don't specify -Fd, a pdb name will be automatically generated.
        // Use this file name to save the pdb so that PIX can find it quickly.
        _wfopen_s(&fp, pPDBName->GetStringPointer(), L"wb");
        fwrite(pPDB->GetBufferPointer(), pPDB->GetBufferSize(), 1, fp);
        fclose(fp);
    }

    //
    // Print hash.
    //
    CComPtr<IDxcBlob> pHash = nullptr;
    pResults->GetOutput(DXC_OUT_SHADER_HASH, IID_PPV_ARGS(&pHash), nullptr);
    if (pHash != nullptr)
    {
        wprintf(L"Hash: ");
        DxcShaderHash* pHashBuf = (DxcShaderHash*)pHash->GetBufferPointer();
        for (int i = 0; i < _countof(pHashBuf->HashDigest); i++)
            wprintf(L"%.2x", pHashBuf->HashDigest[i]);
        wprintf(L"\n");
    }

    //
    // Demonstrate getting the hash from the PDB blob using the IDxcUtils::GetPDBContents API
    //
    CComPtr<IDxcBlob> pHashDigestBlob = nullptr;
    CComPtr<IDxcBlob> pDebugDxilContainer = nullptr;
    if (SUCCEEDED(pUtils->GetPDBContents(pPDB, &pHashDigestBlob, &pDebugDxilContainer)))
    {
        // This API returns the raw hash digest, rather than a DxcShaderHash structure.
        // This will be the same as the DxcShaderHash::HashDigest returned from
        // IDxcResult::GetOutput(DXC_OUT_SHADER_HASH, ...).
        wprintf(L"Hash from PDB: ");
        const BYTE *pHashDigest = (const BYTE*)pHashDigestBlob->GetBufferPointer();
        assert(pHashDigestBlob->GetBufferSize() == 16); // hash digest is always 16 bytes.
        for (int i = 0; i < pHashDigestBlob->GetBufferSize(); i++)
            wprintf(L"%.2x", pHashDigest[i]);
        wprintf(L"\n");

        // The pDebugDxilContainer blob will contain a DxilContainer formatted
        // binary, but with different parts than the pShader blob retrieved
        // earlier.
        // The parts in this container will vary depending on debug options and
        // the compiler version.
        // This blob is not meant to be directly interpreted by an application.
    }

    //
    // Get separate reflection.
    //
    CComPtr<IDxcBlob> pReflectionData;
    pResults->GetOutput(DXC_OUT_REFLECTION, IID_PPV_ARGS(&pReflectionData), nullptr);
    if (pReflectionData != nullptr)
    {
        // Optionally, save reflection blob for later here.

        // Create reflection interface.
        DxcBuffer ReflectionData;
        ReflectionData.Encoding = DXC_CP_ACP;
        ReflectionData.Ptr = pReflectionData->GetBufferPointer();
        ReflectionData.Size = pReflectionData->GetBufferSize();

        CComPtr< ID3D12ShaderReflection > pReflection;
        pUtils->CreateReflection(&ReflectionData, IID_PPV_ARGS(&pReflection));

        // Use reflection interface here.
        
    }
}
```