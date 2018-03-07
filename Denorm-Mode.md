# Denorm Mode

Starting from Shader Model 6.2, you can provide -denorm command line option to specify behaviors for 32-bit floating point denormal number.

Available values for this option are "preserve", "ftz", and "any". These options will be represented as function attributes in DXIL.

Affected instructions for this mode are basic arithmetic operations (add, subtract, multiply, and divide).

Note that fp16/fp64 need to preserve denorms regardless of this option.

| -denorm \<value\>    | function attribute          | behavior                                      |
| :------------------: |:---------------------------:| :-------------------------------------------: |
| preserve             |"fp32-denorm-mode"="preserve"| denorm input/output should be preserved       |
| ftz                  |"fp32-denorm-mode"="ftz"     | denorm input/output should be flushed to zero |
| any                  |"fp32-denorm-mode"="any"     | any behavior is allowed                       |

