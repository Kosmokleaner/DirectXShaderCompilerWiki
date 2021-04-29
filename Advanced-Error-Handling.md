The results of `IDxcCompiler::Compile` can take a few different forms based on the `HRESULT` returned, the status of the `IDxcResult` or, in extreme cases, a thrown structured exception.

## Successful HRESULT

If the `HRESULT` is `S_OK`, the compiler object and allocator are in a good state and can be reused. It's still possible that the compilation failed. To determine if the compilation failed, check the status of the `IDxcResult` and the additional information in the error buffer.

## Failure HRESULT

There are a few ways that `Compile()` might return a failure code. In any of these cases, the process should be terminated as the compiler and allocator cannot be relied on.

* `E_INVALIDARG` - One or more of the parameters to Compile() are incorrect
* `E_OUTOFMEMORY` - An internal system memory allocation failed
* `E_FAIL` - Unexpected c++ exception caught. Indicates an internal compiler error

## Internal Compiler Errors
Occasionally, a shader compilation may encounter bugs in the compiler and produce an internal error. When such errors are encountered, compilation must fail. How this error is reported depends on the severity of the error and how the compiler is being invoked.

### Recoverable
Most fatal internal errors are recoverable. In this case Compile will return `S_OK` and the error log will contain messages prefixed with "Internal Error". If invoked from the command line, these errors will be printed to the output stream.

Examples of recoverable errors:

* incompatible pointer type cast
* unreachable code executed
* assert failure (debug built compilers only)
* explicitly raised fatal errors

### Non-recoverable
Unrecoverable internal errors involve state that cannot reliably write to the error log. In such cases, the compiler will raise a structured exception that can be caught using __try and __except statements around the API call. https://docs.microsoft.com/en-us/cpp/cpp/try-except-statement. This may require moving these calls into an individual dedicated function. The dxc command line executable catches these exceptions by declaring an unhandled exception handler and printing as much information about the failure as possible.


Examples of unrecoverable errors:

* Memory access violation
* Stack overflow


## Example code
Example usage of __try/__except to catch unrecoverable errors.  Assume omitted code as described in [Using dxc.exe and dxcompiler.dll](Using-dxc.exe-and-dxcompiler.dll)

```c++
  int filter(unsigned int code, struct _EXCEPTION_POINTERS *pExceptionInfo) {
    static char scratch[32];
    // report all errors with fputs to prevent any allocation
    if (code == EXCEPTION_ACCESS_VIOLATION) {
      // use pExceptionInfo to document and report error
      fputs("access violation. Attempted to ", stderr);
      if (pExceptionInfo->ExceptionRecord->ExceptionInformation[0])
        fputs("write", stderr);
      else
        fputs("read", stderr);
      fputs(" from address ", stderr);
      sprintf_s(scratch, _countof(scratch), "0x%p\n", 
                (void*)pExceptionInfo->ExceptionRecord->ExceptionInformation[1]);
      fputs(scratch, stderr);
      return EXCEPTION_EXECUTE_HANDLER;
    }
    if (code == EXCEPTION_STACK_OVERFLOW) {
      // use pExceptionInfo to document and report error
      fputs("stack overflow\n", stderr);
      return EXCEPTION_EXECUTE_HANDLER;
    }
    fputs("Unrecoverable Error ", stderr);
    sprintf_s(scratch, _countof(scratch), "0x%08x\n", code);
    fputs(scratch, stderr);
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
          // UNRECOVERABLE ERROR!
          // At this point, state could be extremely corrupt. Terminate the process
          return E_FAIL;
      }
  }

  int main() {
    ...
    //
    // Compile it with specified arguments.
    //
    CComPtr<IDxcResult> pResults;
    if (FAILED(Compile(pCompiler, &Source, pszArgs, _countof(pszArgs), pIncludeHandler, &pResults)))
    {
      // Either an unrecoverable error exception was caught or a failing HRESULT was returned
      // Use fputs to prevent any chance of new allocations
      // Terminate the process
      puts("Internal error or API misuse! Compile Failed\n");
      return 1;
    }

    //
    // Print errors if present.
    //
    CComPtr<IDxcBlobUtf8> pErrors = nullptr;
    // Note that d3dcompiler would return null if no errors or warnings are present.
    // IDxcCompiler3::Compile will always return an error buffer,
    // but its length will be zero if there are no warnings or errors.
    if (SUCCEEDED(pResults->GetOutput(DXC_OUT_ERRORS, IID_PPV_ARGS(&pErrors), nullptr)) &&
        pErrors != nullptr && pErrors->GetStringLength() != 0)
        wprintf(L"Warnings and Errors:\n%S\n", pErrors->GetStringPointer());

    //
    // Quit if the compilation failed.
    //
    HRESULT hrStatus;
    if (FAILED(pResults->GetStatus(&hrStatus)) || FAILED(hrStatus))
    {
        // Compilation failed, but successful HRESULT was returned.
        // Could reuse the compiler and allocator objects. For simplicity, exit here anyway
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
