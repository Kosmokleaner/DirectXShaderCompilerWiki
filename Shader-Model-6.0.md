Shader Model 6.0 is the first shader model that can be compiled with the DirectX Shader Compiler.

The main additions are as follows:

* [[Wave Intrinsics]]. These operations allow wave lanes running in lockstep to communicate, with the goal of abstracting the number of lanes per wave. Drivers are not required to implement this feature for SM6.0.
* 64-bit integers. These are primitive types declared as int64_t and uint64_t, possibly with a vector count suffix, eg uint64_t4. SM 6.0 supports these only for arithmetic and computation, not for memory operations. Drivers are not required to implement this feature for SM6.0.
