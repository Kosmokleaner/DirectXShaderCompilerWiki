# Float16

Starting with Shader Model 6.2, we are introducing true 16-bit scalar types with option /enable-16bit-types. If this mode is enabled, every min precision type is disabled and implicitly converted to its full scalar types.

# DXIL scalar type

Along with 16-bit scalar types in DXIL, we introduced new keywords for scalars in HLSL to map these values. New type mapping can be summarized in these two rules
 - enable-16bit-types are only allowed in -HV 2018 (default)
 - min precision types will map to its exact precision types (e.g min16float -> float16_t) in -eanble-16bit-types mode
 - Fixed data width types are allowed starting in HLSL 2018
 - half will be mapped to float16_t on -enable-16bit-types mode

Below is the table of HLSL Name to DXIL type mappings for each HLSL level and modes.

| HLSL Name     | -HV [<=2017]          | -HV 2018             | [-HV 2018] -enable-16bit-types -T *s_6_2 |
| :-----------: | :--------------------:| :-------------------:|:----------------------------------------:|
| float         | float32_t             | float32_t            | float32_t                                |
| float32_t     | N/A                   | float32_t            | float32_t                                |
| min10float    | min16float(warning)   | min16float(warning)  | float16_t(warning)                       |
| min16float    | min16float            | min16float           | float16_t(warning)                       |
| half          | float32_t             | float32_t            | float16_t                                |
| float16_t     | N/A                   | N/A                  | float16_t                                |
| double        | float64_t             | float64_t            | float64_t                                |
| float64_t     | N/A                   | float64_t            | float64_t                                |
| int           | int32_t               | int32_t              | int32_t                                  |
| int32_t       | N/A                   | int32_t              | int32_t                                  |
| uint          | uint32_t              | uint32_t             | uint32_t                                 |
| uint32_t      | N/A                   | uint32_t             | uint32_t                                 |
| min12int      | min16int(warning)     | min16int(warning)    | int16_t(warning)                         |
| min16int      | min16int              | min16int             | int16_t(warning)                         |
| int16_t       | N/A                   | N/A                  | int16_t                                  |
| min12uint     | min16uint(warning)    | min16uint(warning)   | uint16_t(warning)                        |
| min16uint     | min16uint             | min16uint            | uint16_t(warning)                        |
| uint16_t      | N/A                   | N/A                  | uint16_t                                 |
| int64_t       | int64_t               | int64_t              | int64_t                                  |
| uint64_t      | uint64_t              | uint64_t             | uint64_t                                 |

# Legacy Constant Buffer Alignment

Legacy Constant buffer still contains legacy 4 DWORD (128 bit) row alignment for indexing. For 16bit types, we can fit up to 8 scalars for a single row.

# Structure Alignment

HLSL Structure will follow natural alignment for scalar types. This is equivalent to the layout that C++ compiler would produce under #pragma pack (8).

# Byte Address Buffer

Currently ByteAddressBuffer can only load 'uint' scalar type. Starting with shader model 6.2, we will support templated Load methods. We are only supporting scalar types, or vector of scalars up to 4 components (2 for 8 byte scalars for now). 

    ByteAddressBuffer buffer;

    float f1 = buffer.Load<float>(idx);
    half2 h2 = buffer.Load<half2>(idx);
    uint16_t4 i4 = buffer.Load<uint16_t4>(idx);

# Type Buffer/Texture

No changes in syntax. You can now have a typed buffer of 16 bits (e.g Buffer<half4>)
    
# Atomic Operations

No atomic operations for float16 are supported.

# Intrinsics and Arithmetic Operations

All intrinsics and arithmetic operations that supported min16float/min16int previously will support float17/int16 types. For float16 operations, denormal numbers must be preserved.

# DXIL Changes
- For signature packing, we are packing signature elements based on the width of the scalar type. This means int16/float16 types will never be packed with other scalar types of 4 bytes.
Similar to the previous rule, each row for signature element still contains 4 elements max, regardless of size. We are also still constraining on the total number of rows to be 32 for now. 
This can be changed in the future if people find this limit to be an issue.
- For ByteAddressBuffer and Structured Buffer, Load/Store operations will now map to rawBufferLoad/rawBufferStore with masks to specify which channels need to be loaded. You can differentiate typed buffer load from raw buffer load from opcode.

# Metadata Change

/enable-16bit-types option will map to UseNativeBit16 in shader flag.
