DirectXShaderCompiler has added new versions of `Load` and `Store` methods to the existing `ByteAddressBuffer` and `RWByteAddressBuffer` objects.  The new methods are templated to allow types other than uint, such as aggregate types, to be used directly.

The layout of the type mapped into the buffer matches the layout used for `StructuredBuffer`.  The byteOffset should be aligned by size of the largest component that the type contains.

The methods are defined like so:

```C++
class ByteAddressBuffer {
  ...
  template<typename _T>
  _T Load(in uint byteOffset);

  template<typename _T>
  _T Load(in uint byteOffset, out uint status);
};


class RWByteAddressBuffer {
  ...
  template<typename _T>
  _T Load(in uint byteOffset);

  template<typename _T>
  _T Load(in uint byteOffset, out uint status);

  template<typename _T>
  void Store(in uint byteOffset, in _T value);
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