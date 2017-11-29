Shader Model 6.2 is a superset of [[Shader Model 6.1]].

The main additions are as follows:
* [[16 Bit Scalar Types]]. As opposed to minfloat16, min16int, and min16uint, these values can be used with memory operations (that is, reading and writing via resources or groupshared memory).
* [[Denorm Mode]]. This allows a shader to select the behavior with respect to denormal values: preserv them, flush them to zero, or pick the platform's preferred behavior (typically the most performant).
