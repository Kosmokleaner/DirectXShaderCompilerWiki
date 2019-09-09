```
test_root
├───compiler_pass_tests
│   ├───dxilgen
│   ├───dxil_array_initializer
│   ├───dxil_cleanup_addrspacecast
│   ├───dxil_eliminate_output_dynamic
│   ├───dxil_preserve_all_outputs
│   ├───globalopt
│   ├───gvn
│   ├───instcombine
│   ├───legalize_sample_offset
│   ├───mem2reg_hlsl
│   ├───sccp
│   ├───simple_gvn_hoists
│   └───sroa_hlsl
├───crashes
├───meta_tests
├───scenario_based_tests
│   ├───d3dreflect
│   ├───pix
│   ├───rov
│   └───samples
│       ├───d3d11
│       ├───d3d12
│       └───MiniEngine
├───shader_stage_tests
│   ├───geometry
│   │   └───streamoutputs
│   ├───hull
│   ├───library
│   │   └───lib_arg_flatten
│   ├───mesh
│   │   └───val
│   ├───pixel
│   │   └───earlydepthstencil
│   └───raytracing
│       └───rayquery
└───unit_tests
    ├───classes
    ├───compile_options
    │   ├───pack_optimized
    │   ├───pack_prefix_stable
    │   └───Qstrip_reflect
    ├───control_flow
    │   ├───discard
    │   ├───if_else
    │   │   └───attributes
    │   │       ├───branch
    │   │       └───flatten
    │   ├───loops
    │   │   └───attributes
    │   │       └───unroll
    │   └───switch
    │       └───attributes
    │           └───call
    ├───datalayout
    ├───debug_info
    │   ├───dilocation
    │   ├───locals
    │   ├───misc
    │   ├───shader_params
    │   └───types
    ├───diagnostics
    │   ├───errors
    │   └───warnings
    ├───functions
    │   ├───argument_passing
    │   ├───attribute
    │   ├───entrypoints
    │   ├───misc
    │   ├───overloading
    │   ├───recursion
    │   └───return_type
    │       └───array
    ├───include_handler
    │   └───inc
    ├───intrinsics
    │   ├───abs
    │   ├───acos
    │   ├───AddUint64
    │   ├───all
    │   ├───AllMemoryBarrier
    │   ├───asfloat
    │   ├───asin
    │   ├───asuint
    │   ├───atan
    │   ├───atan2
    │   ├───clip
    │   ├───cos
    │   ├───cosh
    │   ├───countbits
    │   ├───D3DCOLORtoUBYTE4
    │   ├───determinant
    │   ├───DeviceMemoryBarrier
    │   ├───dot
    │   ├───dot2add
    │   ├───dot4add
    │   ├───EvaluateAttributeAtCentroid
    │   ├───EvaluateAttributeAtSample
    │   ├───exp
    │   ├───faceforward
    │   ├───firstbithigh
    │   ├───firstbitlow
    │   ├───fma
    │   ├───frac
    │   ├───frexp
    │   ├───GetAttributeAtVertex
    │   ├───GroupMemoryBarrierWithGroupSync
    │   ├───InterlockedAtomicOps
    │   ├───isfinite
    │   ├───lit
    │   ├───log
    │   ├───mad
    │   ├───max
    │   ├───min
    │   ├───modf
    │   ├───mul
    │   ├───normalize
    │   ├───pow
    │   ├───rcp
    │   ├───reversebits
    │   ├───round
    │   ├───rsqrt
    │   ├───saturate
    │   ├───sin
    │   ├───sinh
    │   ├───smoothstep
    │   ├───sqrt
    │   ├───tan
    │   ├───tanh
    │   ├───transpose
    │   └───WaveOps
    ├───linker
    ├───namespace
    ├───operators
    │   ├───binary
    │   ├───swizzle
    │   └───unary
    ├───preprocessor
    │   └───line_directive
    ├───resource_binding
    ├───root_signature
    ├───semantics
    │   ├───legacy
    │   ├───sv_barycentrics
    │   ├───sv_clipdistance
    │   ├───sv_depth
    │   ├───sv_dispatchthreadid
    │   ├───sv_isfrontface
    │   ├───sv_shadingrate
    │   ├───sv_vertexid
    │   └───sv_viewid
    ├───signature
    ├───types
    │   ├───array
    │   ├───boolean
    │   ├───cast
    │   │   └───identical_layout_structs
    │   ├───conversions
    │   ├───enum
    │   ├───half
    │   ├───matrix
    │   ├───minprecision
    │   ├───modifiers
    │   │   ├───center
    │   │   ├───const
    │   │   ├───global
    │   │   ├───globallycoherent
    │   │   ├───groupshared
    │   │   ├───matrix_packing
    │   │   ├───precise
    │   │   ├───static
    │   │   └───unorm_snorm
    │   ├───objects
    │   │   ├───AppendStructuredBuffer
    │   │   ├───Buffer
    │   │   ├───ByteAddressBuffer
    │   │   ├───Cbuffer
    │   │   ├───CbufferLegacy
    │   │   ├───FeedbackTexture
    │   │   ├───StructuredBuffer
    │   │   ├───Tbuffer
    │   │   └───Texture
    │   ├───struct
    │   ├───typedef
    │   └───vector
    └───validation
```