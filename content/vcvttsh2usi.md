#VCVTTSH2USI
**Convert with Truncation Low FP**

| Instruction En Bit Mode Flag Support Instruction En Bit Mode Flag Support 64/32 CPUID Feature Instruction En Bit Mode Flag CPUID Feature Instruction En Bit Mode Flag Op/ 64/32 CPUID Feature Instruction En Bit Mode Flag 64/32 CPUID Feature Instruction En Bit Mode Flag CPUID Feature Instruction En Bit Mode Flag Op/ 64/32 CPUID Feature |     | Support |             | Description                                                                                                            |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- | ------- | ----------- | ---------------------------------------------------------------------------------------------------------------------- |
| EVEX.LLIG.F3.MAP5.W0 78 /r VCVTTSH2USI r32, xmm1/m16 {sae}                                                                                                                                                                                                                                                                                     | A   | V/V1    | AVX512-FP16 | Convert FP16 value in the low element of xmm1/m16 to an unsigned integer and store the result in r32 using truncation. |
| EVEX.LLIG.F3.MAP5.W1 78 /r VCVTTSH2USI r64, xmm1/m16 {sae}                                                                                                                                                                                                                                                                                     | A   | V/N.E.  | AVX512-FP16 | Convert FP16 value in the low element of xmm1/m16 to an unsigned integer and store the result in r64 using truncation. |

> 1. Outside of 64b mode, the EVEX.W field is ignored. The instruction behaves as if W=0 was used.

## Instruction Operand Encoding

| Op/En | Tuple  | Operand 1     | Operand 2     | Operand 3 | Operand 4 |
| ----- | ------ | ------------- | ------------- | --------- | --------- |
| A     | Scalar | ModRM:reg (w) | ModRM:r/m (r) | N/A       | N/A       |

### Description

This instruction converts the low FP16 element in the source operand to an unsigned integer in the destination general purpose register.

When a conversion is inexact, a truncated (round toward zero) value is returned. If a converted result cannot be represented in the destination format, the floating-point invalid exception is raised, and if this exception is masked, the integer indefinite value is returned.

### Operation

#### VCVTTSH2USI dest, src

```
IF 64-mode and OperandSize == 64:
    DEST.qword := Convert_fp16_to_unsigned_integer64_truncate(SRC.fp16[0])
ELSE:
    DEST.dword := Convert_fp16_to_unsigned_integer32_truncate(SRC.fp16[0])

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
VCVTTSH2USI unsigned int _mm_cvtt_roundsh_u32 (__m128h a, int sae);

```

```
VCVTTSH2USI unsigned __int64 _mm_cvtt_roundsh_u64 (__m128h a, int sae);

```

```
VCVTTSH2USI unsigned int _mm_cvttsh_u32 (__m128h a);

```

```
VCVTTSH2USI unsigned __int64 _mm_cvttsh_u64 (__m128h a);

```

### SIMD Floating-Point Exceptions

Invalid, Precision.

### Other Exceptions

EVEX-encoded instructions, see Table 2-48, “Type E3NF Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
