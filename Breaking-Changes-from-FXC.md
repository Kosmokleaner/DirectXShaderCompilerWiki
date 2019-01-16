**Removed features**
- Derivative expressions (backtick operator)
- Interfaces
- Features supporting the effects framework
- Strings as a primitive type
- Printf/assert

**Other known incompatibilities (not considered bugs)**
- Shader entry point must be in the global namespace (`/E foo::main` is unsupported)
- `#pragma pack_matrix` does not apply at the exact same token-level granularity
- Struct definitions cannot declare functions (`struct Foo {} func() {}` is unsupported)