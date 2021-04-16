Occasionally, a shader compilation may encounter bugs in the compiler and produce an internal error. When such errors are encountered, compilation must fail. How this error is reported depends on the severity of the error and how the compiler is being invoked.

Most fatal internal errors are recoverable. In this case the compiler can retrieve details about the error and return the information in the error log and return an error code. This is true whether the compiler is invoked from the API or from the command line.

Examples of recoverable errors:

* incompatible pointer type cast
* unreachable code executed
* assert failure (debug built compilers only)
* explicitly raised fatal errors


Unrecoverable internal errors involve state that cannot reliably write to the error log. In such cases, the compiler will raise a structured exception that can be caught using __try and __except statements around the API call. https://docs.microsoft.com/en-us/cpp/cpp/try-except-statement. This may require moving these calls into an individual dedicated function. The dxc command line executable catches these exceptions by declaring an unhandled exception handler and printing as much information about the failure as possible.


Examples of unrecoverable errors:

* Memory access violation
* Stack overflow

Example usage of __try/__except to catch unrecoverable errors.  Assume omitted code as described in [Using dxc.exe and dxcompiler.dll](Using-dxc.exe-and-dxcompiler.dll)

```c++
  int filter(unsigned int code, struct PEXCEPTION_POINTERS pExceptionInfo) {
    if (code == EXCEPTION_ACCESS_VIOLATION) {
      // use pExceptionInfo to document and report error
      return EXCEPTION_EXECUTE_HANDLER;
    }
    if (code == EXCEPTION_STACK_OVERFLOW) {
      // use pExceptionInfo to document and report error
      return EXCEPTION_EXECUTE_HANDLER;
    }
    return EXCEPTION_CONTINUE_SEARCH;
  }

  HRESULT Compile(IDxcCompiler3 *pCompiler, DxcBuffer *pSource, LPCWSTR pszArgs[],
             int argCt, IDxcIncludeHandler *pIncludeHandler, IDxcResult **pResults) {

      __try {
          return pCompiler->Compile(
                   pSource,                // Source buffer.
                   pszArgs,                // Array of pointers to arguments.
                   argCt,                  // Number of arguments.
                   pIncludeHandler,        // User-provided interface to handle #include directives (optional).
                   IID_PPV_ARGS(pResults) // Compiler output status, buffer, and errors.
          );
      } __except(filter(GetExceptionCode(), GetExceptionInformation())) {
          // Report unrecoverable internal error
          return 1;
      }
  }

  int main() {
    ...
    CComPtr<IDxcResult> pResults;
    if (FAILED(Compile(pCompiler, &Source, pszArgs, _countof(pszArgs), pIncludeHandler, &pResults)))
      return 1;
    
    // Recoverable error handling
    CComPtr<IDxcBlobUtf8> pErrors = nullptr;
    if (FAILED(pResults->GetOutput(DXC_OUT_ERRORS, IID_PPV_ARGS(&pErrors), nullptr)))
      return 1;
    if (pErrors != nullptr && pErrors->GetStringLength() != 0)
      wprintf(L"Warnings and Errors:\n%S\n", pErrors->GetStringPointer());

    //
    // Quit if the compilation failed.
    //
    HRESULT hrStatus;
    if (FAILED(pResults->GetStatus(&hrStatus)) || FAILED(hrStatus))
    {
        wprintf(L"Compilation Failed\n");
        return 1;
    }

    //
    // Save shader binary.
    //
    CComPtr<IDxcBlob> pShader = nullptr;
    CComPtr<IDxcBlobUtf16> pShaderName = nullptr;
    if (SUCCEEDED(pResults->GetOutput(DXC_OUT_OBJECT, IID_PPV_ARGS(&pShader), &pShaderName)) &&
        pShader != nullptr)
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
    if(SUCCEEDED(pResults->GetOutput(DXC_OUT_PDB, IID_PPV_ARGS(&pPDB), &pPDBName)))
    {
        FILE* fp = NULL;

        // Note that if you don't specify -Fd, a pdb name will be automatically generated. Use this file name to save the pdb so that PIX can find it quickly.
        _wfopen_s(&fp, pPDBName->GetStringPointer(), L"wb");
        fwrite(pPDB->GetBufferPointer(), pPDB->GetBufferSize(), 1, fp);
        fclose(fp);
    }

    //
    // Print hash.
    //
    CComPtr<IDxcBlob> pHash = nullptr;
    if (SUCCEEDED(pResults->GetOutput(DXC_OUT_SHADER_HASH, IID_PPV_ARGS(&pHash), nullptr)) &&
        pHash != nullptr)
    {
        wprintf(L"Hash: ");
        DxcShaderHash* pHashBuf = (DxcShaderHash*)pHash->GetBufferPointer();
        for (int i = 0; i < _countof(pHashBuf->HashDigest); i++)
            wprintf(L"%x", pHashBuf->HashDigest[i]);
        wprintf(L"\n");
    }


    //
    // Get separate reflection.
    //
    CComPtr<IDxcBlob> pReflectionData;
    if (SUCCEEDED(pResults->GetOutput(DXC_OUT_REFLECTION, IID_PPV_ARGS(&pReflectionData), nullptr)) &&
        pReflectionData != nullptr)
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
