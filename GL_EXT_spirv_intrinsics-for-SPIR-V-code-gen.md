### Overview

[GL_EXT_spirv_intrinsics](https://github.com/KhronosGroup/GLSL/pull/157) is a GLSL language extension to support embedding arbitrary SPIR-V instructions in the middle of the GLSL code similar to the inlined assembly in the C code.
We designed the HLSL version of GL_EXT_spirv_intrinsics to allow developers to embed arbitrary SPIR-V instructions in the HLSL code.

### Contributors

- Jaebaek Seo, Google
- Jiao Lu, AMD
- Tobias Hector, AMD

(Alphabetical order)

### Spec

- vk::ext_execution_mode(uint _execution_mode_, ...);
  - Stand-alone intrinsic function to emit an `OpExecutionMode`.
  - To specify extensions and/or capabilities needed by the `OpExecutionMode`, we have to use `vk::ext_extension(string)` and `vk::ext_capability(uint)` attributes (see below).
  - Return type: void
  - uint _execution_mode_ must be a constant expression.
  - Extra parameters must be constant expressions. Some [execution modes](https://www.khronos.org/registry/SPIR-V/specs/unified1/SPIRV.html#_execution_mode) e.g., `Invocations`, `LocalSize`, .. have extra parameters. Since extra parameters of `OpExecutionMode` are literals, DXC will emit these extra parameters as literals as well.
- `[[vk::ext_extension(string _extension_name_)]]`
  - An attribute to specify `OpExtension` instruction.
  - It can be an attribute of `vk::ext_execution_mode(...)`, `vk::ext_execution_mode_id(...)`, a function declared with `vk::ext_instruction(...)`, or a function declared with `[[vk::ext_type_def(...)]]`. We can use multiple `[[vk::ext_extension(string extension_name)]]` for a single function.
  - string _extension_name_ must be a constant string.
- `[[vk::ext_capability(uint _capability_)]]`
  - An attribute to specify `OpCapability` instruction.
  - It can be an attribute of `vk::ext_execution_mode(...)`, `vk::ext_execution_mode_id(...)`, a function declared with `vk::ext_instruction(...)`, or a function declared with `[[vk::ext_type_def(...)]]`. We can use multiple `[[vk::ext_extension(string extension_name)]]` for a single function.
  - uint _capability_ must be a constant expression.

Example:
```
// HLSL
[[vk::ext_capability(/* StencilExportEXT */ 5013)]]
[[vk::ext_extension("SPV_EXT_shader_stencil_export")]]
vk::ext_execution_mode(/* StencilRefReplacingEXT */ 5027);

// SPIR-V
OpCapability StencilExportEXT
...
OpExtension "SPV_EXT_shader_stencil_export"
...
OpExecutionMode %main StencilRefReplacingEXT
```

- vk::ext_execution_mode_id(uint _execution_mode_, ...);
  - Stand-alone intrinsic function to emit an `OpExecutionModeId`.
  - To specify extensions and/or capabilities needed by the `OpExecutionModeId`, we have to use `vk::ext_extension(string)` and `vk::ext_capability(uint)` attributes.
  - Return type: void
  - uint _execution_mode_ must be a constant expression.
  - Extra parameters must be constant expressions. Some [execution modes](https://www.khronos.org/registry/SPIR-V/specs/unified1/SPIRV.html#_execution_mode) e.g., `LocalSizeId` have extra id parameters. Since they must be result ids of instructions, DXC will generate `OpConstantXXX` instructions for them.
- `[[vk::ext_instruction(uint _opcode_, string _extended_instruction_set_)]]
return_type mock_function(parameters, …, [[vk::ext_reference]] parameter, … , [[vk::ext_literal]] parameter …);`
  - `[[vk::ext_instruction(uint opcode, string extended_instruction_set)]]`
    - An attribute to specify that the function declaration with this attribute will be used as a SPIR-V instruction.
    - To specify extensions and/or capabilities needed by the SPIR-V instruction, we have to use `vk::ext_extension(string)` and `vk::ext_capability(uint)` attributes.
    - uint _opcode_ must be a constant expression.
    - string _extended_instruction_set_  is optional and it must be a constant string.
  - `[[vk::ext_reference]]`
    - If a parameter has a `[[vk::ext_reference]]` attribute, we use the pointer as the operand of SPIR-V instruction instead of loading it and using the value as the operand.
  - `[[vk::ext_literal]]`
    - If a parameter has an attribute `[[vk::ext_literal]]`, we use it in a literal form as the operand of SPIR-V instruction instead of a result id of a SPIR-V instruction.
  - Parameter without `[[vk::ext_reference]]` and `[[vk::ext_literal]]` attributes
    - If it is a variable, we load it using OpLoad and use the loaded value as the operand of the SPIR-V instruction.
    - If it is a constant, we create OpConstant and use it as the operand.
    - If it is a SPIR-V result id whose type is `vk::ext_result_id<T>` (we will explain `vk::ext_result_id<T>` below), we use the SPIR-V result id as the operand.
  - Return type.
    - If the return type is void, the SPIR-V instruction will not have a result id and a result type.
    - If the return type is not void e.g., int, float, …, the SPIR-V instruction will have a result id. The result type will be the same as the return type.
    - If the caller of the _mock_function_ is used for l-value e.g., variable initialization or function argument passing, we will generate an `OpStore` to store the result of the SPIR-V instruction in the l-value e.g., variable.
    - If the mock_function is used to initialize a variable with `vk::ext_result_id<T>` type, we do not generate an `OpStore`, but set the variable with `vk::ext_result_id<T>` type as the result id of the SPIR-V instruction. In addition, the result type of the SPIR-V instruction will be **T** that is the template argument of vk::ext_result_id<**T**>.
      - It is particularly useful when we want to use a result id of the SPIR-V instruction for the operand of another SPIR-V instruction declared by a mock function with `[[vk::ext_instruction(...)]]` attribute.
- vk::ext_result_id<**T**>
  - A type that can be used only for the result or a parameter of a function with `[[vk::ext_instruction(uint opcode, string extended_instruction_set)]]` attribute
  - A variable defined with vk::ext_result_id<**T**> type does not have a physical storage. It becomes the result id of the SPIR-V instruction whose result type is **T**.

Example:
```
// HLSL
[[vk::ext_capability(/* ShaderClockKHR */ 5055)]]
[[vk::ext_extension("SPV_KHR_shader_clock")]]
[[vk::ext_instruction(5056)]]
uint64_t readClock(uint scope);
...
uint64_t clock = readClock(1);

// SPIR-V
OpCapability ShaderClockKHR
...
OpExtension "SPV_KHR_shader_clock"
...
%clock = OpVariable %_ptr_Function_uint64 Function
%result_id = OpReadClockKHR %uint64 %uint_1
OpStore %clock %result_id


// HLSL
[[vk::ext_instruction(/* OpLoad */ 61)]]
vk::ext_result_id<float> load([[vk::ext_reference]] float pointer,
                         [[vk::ext_literal]] int memoryOperands);

[[vk::ext_instruction(/* OpStore */ 62)]]
void store([[vk::ext_reference]] float pointer,
           vk::ext_result_id<float> value,
           [[vk::ext_literal]] int memoryOperands)
...
float foo, bar;
vk::ext_result_id<float> foo_value = load(foo, /* None */ 0x0);
store(bar, foo_value, /* Volatile */ 0x1);

// SPIR-V
%foo = OpVariable %_ptr_Function_float Function
%bar = OpVariable %_ptr_Function_float Function
%foo_value = OpLoad %float %foo None
OpStore %bar %foo_value Volatile


// HLSL
[[vk::ext_instruction(/* FMin3AMD */ 1,
                      "SPV_AMD_shader_trinary_minmax")]]
float2 FMin3AMD(float2 x, float2 y, float2 z);
...
float2 foo = FMin3AMD(x, y, z);

// SPIR-V
%30 = OpExtInstImport "SPV_AMD_shader_trinary_minmax"
...
%foo = OpVariable %_ptr_Function_float Function
...
         %26 = OpLoad %v2float %x
         %27 = OpLoad %v2float %y
         %28 = OpLoad %v2float %z
         %29 = OpExtInst %v2float %30 FMin3AMD %26 %27 %28
               OpStore %foo %29
```

- `[[vk::ext_type_def(uint _unique_type_id_, uint opcode)]]
void createTypeXYZ(parameters, …, [[vk::ext_reference]] parameter, … , [[vk::ext_literal]] parameter …);`
  - `[[vk::ext_type_def(uint unique_type_id, uint opcode)]]` is an attribute similar to `[[vk::ext_instruction(...)]]`, but it specifies that a function declaration will be used to define a type with opcode.
  - The function declared with `[[vk::ext_type_def(..)]]` must have a void return type.
  - After declaring the function with `[[vk::ext_type_def(..)]]`, we have to call the declared function e.g., void createTypeXYZ(..) with proper arguments to actually define the type.
  - uint _unique_type_id_ can be any constant unsigned integer number, but it must be a unique identifier for the defined type. It will be used by vk::ext_type<uint unique_type_id> (see below).
  - uint opcode must be the opcode of the type instruction.
  - Parameters for the declared function are extra operands to define the type.
- vk::ext_type<uint _unique_type_id_>
  - A special type that specifies the type created by a function declared with `[[vk::ext_type_def(..)]]`.
  - uint _unique_type_id_ must be the same as the one of `[[vk::ext_type_def(uint unique_type_id, uint opcode)]]`.

Example:
```
// HLSL
[[vk::ext_type_def(/* Unique id for type */ 0, /* OpTypeInt */ 21)]]
void createTypeInt([[vk::ext_literal]] int sizeInBits,
                   [[vk::ext_literal]] int signedness);
createTypeInt(/* sizeInBits */ 12, /* signedness */ 0);
...
vk::ext_type</* Unique id for type */ 0> foo = 3;

// SPIR-V
%int_in_12bits = OpTypeInt 12 0
%_ptr_Function_int_in_12bits = OpTypePointer Function %int_in_12bits
%i3_in_12bits = OpConstant %int_in_12bits 3
...
%foo = OpVariable %_ptr_Function_int_in_12bits Function
OpStore %foo %i3_in_12bits


// HLSL
[[vk::ext_type_def(/* Unique id for type */ 0, /* OpTypeInt */ 21)]]
void createTypeInt([[vk::ext_literal]] int sizeInBits,
                   [[vk::ext_literal]] int signedness);
createTypeInt(/* sizeInBits */ 12, /* signedness */ 0);
...
[[vk::ext_type_def(/* Unique id for type */ 1, /* OpTypeVector */ 23)]]
void createTypeVector([[vk::ext_reference]] vk::ext_type<0> typeInt,
                      [[vk::ext_literal]] int componentCount);
createTypeVector(/* typeInt */ {}, /* componentCount */ 3);
...
vk::ext_type</* Unique id for type */ 1> foo = {1, 2, 3};

// SPIR-V
%int_in_12bits = OpTypeInt 12 0
%vint3_in_12bits = OpTypeVector %int_in_12bits 3
%_ptr_Function_vint3_in_12bits = OpTypePointer Function %vint3_in_12bits
%i1_in_12bits = OpConstant %int_in_12bits 1
%i2_in_12bits = OpConstant %int_in_12bits 2
%i3_in_12bits = OpConstant %int_in_12bits 3
%i123_in_12bits = OpConstant %vint3_in_12bits %i1_in_12bits
                                              %i2_in_12bits
                                              %i3_in_12bits
...
%foo = OpVariable %_ptr_Function_vint3_in_12bits Function
OpStore %foo %i123_in_12bits


// HLSL
[[vk::ext_capability(/* RayQueryKHR */ 4472)]]
[[vk::ext_extension("SPV_KHR_ray_query")]]
[[vk::ext_type_def(/* Unique id for type */ 2,
                   /* OpTypeRayQueryKHR */ 4472)]]
void createTypeRayQueryKHR();
createTypeRayQueryKHR();
...
[[vk::ext_instruction(/* OpRayQueryTerminateKHR */ 4474)]]
void rayQueryTerminateEXT(
       [[vk::ext_reference]] vk::ext_type(typeRayQueryKHR) rq);
...
vk::ext_type</* Unique id for type */ 2> rq;
rayQueryTerminateEXT(rq);

// SPIR-V
OpCapability RayQueryKHR
...
OpExtension "SPV_KHR_ray_query"
...
%typeRayQueryKHR = OpTypeRayQueryKHR
...
%rq = OpVariable %_ptr_Function_typeRayQueryKHR Function
OpRayQueryTerminateKHR %rq
```

- `[[vk::ext_storage_class(uint storage_class)]]` type
  - An attribute that must be added to a type to specify the storage class.

Example:
```
// HLSL
[[vk::ext_storage_class(/* CrossWorkgroup */ 5)]] int foo;

// SPIR-V
%foo = OpVariable %_ptr_CrossWorkgroup_int CrossWorkgroup
```

- `[[vk::ext_decorate(decoration, … int / float / bool literal)]]`
  - An attribute for a decoration of a variable. It takes the first integer parameter as its `Decoration` operand, and further parameters as `Extra Operands`. If it is used for a member of a struct/class type, OpMemberDecorate will be generated to decorate the member. Otherwise, OpDecorate will be generated to decorate the variable.
- `[[vk::ext_decorate_id(decoration, … constant expression)]]`
  - Similar to `[[vk::ext_decorate(decoration, … int / float / bool literal)]]` but it generates OpDecorateId
- `[[vk::ext_decorate_string(decoration, … string literals)]]`
  - Similar to `[[vk::ext_decorate(decoration, … int / float / bool literal)]]` but it generates OpDecorateString or OpMemberDecorateString.

### Status

- Github issue tracking overall implementation: [#3919](https://github.com/microsoft/DirectXShaderCompiler/issues/3919)
- Known issues
  - Enable `[[vk::ext_decorate/_id/_string]]` for struct/class fields or function parameter [#4195](https://github.com/microsoft/DirectXShaderCompiler/issues/4195).