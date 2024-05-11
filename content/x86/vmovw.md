# VMOVW

**Move Word**

| Instruction En bit Mode Flag Support Instruction En bit Mode Flag Support 64/32 CPUID Feature Instruction En bit Mode Flag CPUID Feature Instruction En bit Mode Flag Op/ 64/32 CPUID Feature Instruction En bit Mode Flag 64/32 CPUID Feature Instruction En bit Mode Flag CPUID Feature Instruction En bit Mode Flag Op/ 64/32 CPUID Feature |     | Support |             | Description                     |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- | ------- | ----------- | ------------------------------- |
| EVEX.128.66.MAP5.WIG 6E /r VMOVW xmm1, reg/m16                                                                                                                                                                                                                                                                                                 | A   | V/V     | AVX512-FP16 | Copy word from reg/m16 to xmm1. |
| EVEX.128.66.MAP5.WIG 7E /r VMOVW reg/m16, xmm1                                                                                                                                                                                                                                                                                                 | B   | V/V     | AVX512-FP16 | Copy word from xmm1 to reg/m16. |

## Instruction Operand Encoding

| Op/En | Tuple  | Operand 1     | Operand 2     | Operand 3 | Operand 4 |
| ----- | ------ | ------------- | ------------- | --------- | --------- |
| A     | Scalar | ModRM:reg (w) | ModRM:r/m (r) | N/A       | N/A       |
| B     | Scalar | ModRM:r/m (w) | ModRM:reg (r) | N/A       | N/A       |

### Description

This instruction either (a) copies one word element from an XMM register to a general-purpose register or memory location or (b) copies one word element from a general-purpose register or memory location to an XMM register. When writing a general-purpose register, the lower 16-bits of the register will contain the word value. The upper bits of the general-purpose register are written with zeros.

### Operation

#### VMOVW dest, src (two operand load)

```
DEST.word[0] := SRC.word[0]
DEST[MAXVL:16] := 0

```

#### VMOVW dest, src (two operand store)

```
DEST.word[0] := SRC.word[0]
// upper bits of GPR DEST are zeroed

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
VMOVW short _mm_cvtsi128_si16 (__m128i a);

```

```
VMOVW __m128i _mm_cvtsi16_si128 (short a);

```

### SIMD Floating-Point Exceptions

None

### Other Exceptions

EVEX-encoded instructions, see Table 2-57, “Type E9NF Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
