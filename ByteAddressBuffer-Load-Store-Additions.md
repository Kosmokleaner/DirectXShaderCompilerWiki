DirectXShaderCompiler has added new versions of `Load` and `Store` methods to the existing `ByteAddressBuffer` and `RWByteAddressBuffer` objects.  The new methods are templated to allow types other than uint, such as aggregate types, to be used directly.

The layout of the type mapped into the buffer matches the layout used for `StructuredBuffer`.  The `byteOffset` should be aligned by the size (in bytes) of the largest scalar type contained within type `T`.  For instance, if the largest type is uint64_t, `byteOffset` must be aligned by 8.  If the largest type is `float16_t`, then the minimum alignment required is 2.

The methods are defined like so:

```C++
class ByteAddressBuffer {
  ...
  template<typename T>
  T Load(in uint byteOffset);

  template<typename T>
  T Load(in uint byteOffset, out uint status);
};


class RWByteAddressBuffer {
  ...
  template<typename T>
  T Load(in uint byteOffset);

  template<typename T>
  T Load(in uint byteOffset, out uint status);

  template<typename T>
  void Store(in uint byteOffset, in T value);
};
```

Example usage:
```HLSL
ByteAddressBuffer BAB;
RWByteAddressBuffer rwBAB;
...
  MyStruct val = BAB.Load<MyStruct>(offset);
  // Use val ...
  ...
  // Store struct to RWByteAddressBuffer:
  rwBAB.Store(offset, val);
```