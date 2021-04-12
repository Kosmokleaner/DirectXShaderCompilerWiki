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

  int Compile(IDxcCompiler3 *pCompiler, DxcBuffer *pSource, LPCWSTR pszArgs[],
             int argCt, IDxcIncludeHandler *pIncludeHandler, IDxcResult **pResults) {

      __try {
          pCompiler->Compile(
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

    return 0;
  }

  int main() {
    ...
    CComPtr<IDxcResult> pResults;
    int res = Compile(pCompiler, &Source, pszArgs, _countof(pszArgs), pIncludeHandler, &pResults);
    if (res) { return res; }
    
    // Recoverable error handling as indicted in "Using dxc.exe and dxcompiler.dll"
    CComPtr<IDxcBlobUtf8> pErrors = nullptr;
    pResults->GetOutput(DXC_OUT_ERRORS, IID_PPV_ARGS(&pErrors), nullptr);
    if (pErrors != nullptr && pErrors->GetStringLength() != 0)
      wprintf(L"Warnings and Errors:\n%S\n", pErrors->GetStringPointer());

    HRESULT hrStatus;
    pResults->GetStatus(&hrStatus);
    if (FAILED(hrStatus)) {
      wprintf(L"Compilation Failed\n");
      return 1;
    }
  }
```
