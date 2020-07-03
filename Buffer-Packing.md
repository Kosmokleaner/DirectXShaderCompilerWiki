# Alignment and Buffer Packing

## Basic Type Alignment and Size

Besides other packing rules that impose additional alignment constraints,
particularly in the case of legacy constant buffer packing,
here are the basic size and alignment rules for HLSL types stored in buffers.

Types with non-obvious storage sizes:

  1. bool storage is 32-bits
  1. In min-precision mode
    (default when compiled without -enable-16bit-types argument):
      - min-precision types have storage size of 32-bits
      - HLSL half type maps to full 32-bit float type
  1. In native 16-bit mode (compiled with -enable-16bit-types):
      - min-precision types map to native 16-bit types
      - HLSL half type maps to native 16-bit float16_t type
      - native 16-bit types have storage size of 16-bits (as expected)
  1. Doubles and 64-bit ints have
    a storage size (and alignment) of 64-bits (8 bytes)
  1. Aggregate (struct/array) sizes depend on
    additional packing and alignment rules specific to the buffer type.

Basic Type Alignment rules - additional alignment and layout rules
must be applied on top of these for legacy constant buffers:

  1. Scalars are naturally aligned to their storage size.
  1. Vector and Matrix types are aligned by their component type alignment.
  1. Structs are aligned by the max of it's member alignments.
  1. Arrays are aligned by the alignment of the array element type.

Matrix types may be layed out in either row-major or column-major orientation.
This impacts not only component ordering in memory,
but padding for row (or column) alignment in legacy constant buffers.
For storage, the column major matrix can be considered equivalent
to a row major matrix with the dimensions transposed.
In other words, `row_major matrix<float, R, C>` is equivalent to
`column_major matrix<float, C, R>` in storage layout and alignment constraints.
Therefore the alignment constraints will be described here
in terms of the row_major equivalent storage layout.

## Constant Buffer Packing

### "Legacy" Constant Buffer Basics (excluding min-precision or 16-bit types)

  1. The Constant Buffer is arranged like an array of 16-byte rows.
  1. Certain basic types will be packed into the last used row
    if they fit into the available space,
    starting at the next available aligned position.
    If they do not fit, they will start a new 16-byte aligned row.
      - Scalars
      - Vectors
      - Matrix with only one row in row_major equivalent storage layout
  1. Matrix types with more than one row in row_major equivalent storage layout
    are aligned to the 16-byte row.
    Each row in storage layout (each column for column_major matrix)
    is aligned to a 16-byte row.
  1. Aggregates (arrays and structures) are always 16-byte row-aligned.
    If the aggregate ends in an element that does not completely fill a row,
    the remaining space on the last row may be used by the next value.
  1. Structure members follow these same rules.
    Because structures are always 16-byte row-aligned,
    the offset layout within the structure (for use in legacy constant buffers)
    is independent of the particular structure instance location.

### "Legacy" Layout in min-precision mode (default)

In addition to the rules above, min-precision values introduce extra packing constraints.
A row with min-precision values cannot also contain fixed precision values
(such as float, bool, half, uint, double, uint64_t, etc...).
Therefore, no min-precision value will be packed into the remaining space on the last row
if the last row contains fixed precision values,
and no fixed precision value will be packed into a row containing min-precision values.
In each case, a new row is started.

### "Legacy" Layout in native 16-bit types mode (-enable-16bit-types)

Native 16-bit types may be packed into the same rows as other fixed precision types.
Alignment and other rules stated above all apply still.
For instance, a 32-bit value following a single 16-bit value starting a row
will be 32-bit aligned, leaving 16-bits of padding between the values.

### "Non-Legacy" packing (-no-legacy-cbuf-layout)

Follows Structured Buffer Packing.

## Structured Buffer Packing

Structured Buffers follow the rules under Basic Type Alignment and Size
without the additional padding or alignment constraints for legacy constant buffers.

## ByteAddressBuffer templated Load/Store operations

ByteAddressBuffer in DXC supports templated Load/Store operations that allow you
to load or store a structure at a byte offset location.
The layout of this structure is identical to that used for Structured Buffer packing, starting at the given offset.
One wrinkle is that the actual absolute alignment of the structure elements
are not greater than the alignment guaranteed by
the base address of the view plus the byte offset given to the operation.
