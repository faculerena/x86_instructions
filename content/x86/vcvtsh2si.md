#VCVTSH2SI
**Convert Low FP**

| Instruction En Bit Mode Flag Support Instruction En Bit Mode Flag Support 64/32 CPUID Feature Instruction En Bit Mode Flag CPUID Feature Instruction En Bit Mode Flag Op/ 64/32 CPUID Feature Instruction En Bit Mode Flag 64/32 CPUID Feature Instruction En Bit Mode Flag CPUID Feature Instruction En Bit Mode Flag Op/ 64/32 CPUID Feature |     | Support |             | Description                                                                               |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- | ------- | ----------- | ----------------------------------------------------------------------------------------- |
| EVEX.LLIG.F3.MAP5.W0 2D /r VCVTSH2SI r32, xmm1/m16 {er}                                                                                                                                                                                                                                                                                        | A   | V/V1    | AVX512-FP16 | Convert the low FP16 element in xmm1/m16 to a signed integer and store the result in r32. |
| EVEX.LLIG.F3.MAP5.W1 2D /r VCVTSH2SI r64, xmm1/m16 {er}                                                                                                                                                                                                                                                                                        | A   | V/N.E.  | AVX512-FP16 | Convert the low FP16 element in xmm1/m16 to a signed integer and store the result in r64. |

> 1. Outside of 64b mode, the EVEX.W field is ignored. The instruction behaves as if W=0 was used.

## Instruction Operand Encoding

| Op/En | Tuple  | Operand 1     | Operand 2     | Operand 3 | Operand 4 |
| ----- | ------ | ------------- | ------------- | --------- | --------- |
| A     | Scalar | ModRM:reg (w) | ModRM:r/m (r) | N/A       | N/A       |

### Description

This instruction converts the low FP16 element in the source operand to a signed integer in the destination general purpose register.

When a conversion is inexact, the value returned is rounded according to the rounding control bits in the MXCSR register or the embedded rounding control bits. If a converted result cannot be represented in the destination format, the floating-point invalid exception is raised, and if this exception is masked, the integer indefinite value is returned.

### Operation

#### VCVTSH2SI dest, src

```
IF *SRC is a register* and (EVEX.b = 1):
    SET_RM(EVEX.RC)
ELSE:
    SET_RM(MXCSR.RC)
IF 64-mode and OperandSize == 64:
    DEST.qword := Convert_fp16_to_integer64(SRC.fp16[0])
ELSE:
    DEST.dword := Convert_fp16_to_integer32(SRC.fp16[0])

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
VCVTSH2SI int _mm_cvt_roundsh_i32 (__m128h a, int rounding);

```

```
VCVTSH2SI __int64 _mm_cvt_roundsh_i64 (__m128h a, int rounding);

```

```
VCVTSH2SI int _mm_cvtsh_i32 (__m128h a);

```

```
VCVTSH2SI __int64 _mm_cvtsh_i64 (__m128h a);

```

### SIMD Floating-Point Exceptions

Invalid, Precision.

### Other Exceptions

EVEX-encoded instructions, see Table 2-48, “Type E3NF Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
