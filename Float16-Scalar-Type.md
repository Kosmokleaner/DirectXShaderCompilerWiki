# Float16

Starting with Shader Model 6.2, we are introducing true 16-bit floating point precision type with option /enable-half. If this mode is enabled, every min precision type is disabled and implicitly converted to its full scalar types.

# Metadata Change

/enable-half option will map to UseNativeHalf in the shader flag.

# DXIL scalar type in Shader Model 6.2

| HLSL          | DXIL       | DXIL with /enable-half |
| :-----------: | :--------: | :---------------------:     |
| min10float    | half       | half                        |
| min16float    | half       | half                        |
| half          | float      | half                        |
| float         | float      | float                       |
| min12int      | i16        | i32                         |
| min16int      | i16        | i32                         |
| int           | i32        | i32                         |



# Legacy Constant Buffer Alignment

Legacy Constant buffer still contains legacy 4 DWORD (128 bit) row alignment for indexing. For float16, we can fit up to 8 halfs for a single row.

# Structure Alignment

HLSL Structure will follow natural alignment for scalar types. This is equivalent to the layout that C++ compiler would produce under #pragma pack (8).

# Byte Address Buffer

Currently byte address buffer can only load 'uint' scalar type. Starting with shader model 6.2, we will support templated Load methods to allow loading types like the following for scalars and vectors:

    float f1 = buffer.Load<float>(idx);
    half4 h1 = buffer.Load<half4>(idx);

This change not only applies to half types, but other types as well.

# Type Buffer

No changes in syntax. Half type will be interpreted as 16 bit scalars in memory.
    
# Atomic Operations

No atomic operations for float16 are supported.

# Intrinsics

All intrinsics that supported min16float previously will support half types.
