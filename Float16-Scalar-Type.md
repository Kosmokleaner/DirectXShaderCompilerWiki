# Float16

Starting with Shader Model 6.2, we are introducing true 16-bit floating point precision type with option /enable-half. If this mode is enabled, every min precision type is disabled and implicitly converted to its full scalar types.

# DXIL scalar type in Shader Model 6.2

| HLSL          | DXIL       | DXIL with /enable-half      |
| :-----------: | :--------: | :------------------------:  |
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

Currently byte address buffer can only load 'uint' scalar type. Starting with shader model 6.2, we will support other Load methods to allow loading types like the following for scalars and vectors:

    float f1 = buffer.LoadFloat(idx);
    half4 h4 = buffer.LoadHalf4(idx);

This change applies to half, float, int, and double scalar types.

# Type Buffer

No changes in syntax. Half type will be interpreted as 16 bit scalars in memory.
    
# Atomic Operations

No atomic operations for float16 are supported.

# Intrinsics and Arithmetic Operations

All intrinsics and arithmetic operations that supported min16float previously will support half types. Denormal numbers must be preserved.

# DXIL Changes
- For signature packing, we are packing signature elements based on the width of the scalar type. This means float16 will never be packed with other scalar types of 4 bytes.
Similar to the previous rule, each row for signature element still contains 4 elements max, regardless of size. We are also still constraining on the total number of rows to be 32 for now. 
This can be changed in the future if people find this limit to be an issue.
- For ByteAddressBuffer and Structured Buffer, Load operations will now map to rawBufferLoad with masks to specify which channels need to be loaded. You can differentiate typed buffer load from raw buffer load from opcode.

# Metadata Change

/enable-half option will map to UseNativeHalf in shader flag.
