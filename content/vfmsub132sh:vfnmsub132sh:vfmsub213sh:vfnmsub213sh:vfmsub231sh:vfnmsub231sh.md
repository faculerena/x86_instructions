# VFMSUB132SH/VFNMSUB132SH/VFMSUB213SH/VFNMSUB213SH/VFMSUB231SH/VFNMSUB231SH

**Fused Multiply**

| Instruction En Bit Mode Flag Support Instruction En Bit Mode Flag Support 64/32 CPUID Feature Instruction En Bit Mode Flag CPUID Feature Instruction En Bit Mode Flag Op/ 64/32 CPUID Feature Instruction En Bit Mode Flag 64/32 CPUID Feature Instruction En Bit Mode Flag CPUID Feature Instruction En Bit Mode Flag Op/ 64/32 CPUID Feature |     | Support |             | Description                                                                                                                                             |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- | ------- | ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| EVEX.LLIG.66.MAP6.W0 9B /r VFMSUB132SH xmm1{k1}{z}, xmm2, xmm3/m16 {er}                                                                                                                                                                                                                                                                        | A   | V/V     | AVX512-FP16 | Multiply FP16 values from xmm1 and xmm3/m16, subtract xmm2, and store the result in xmm1 subject to writemask k1.                                       |
| EVEX.LLIG.66.MAP6.W0 AB /r VFMSUB213SH xmm1{k1}{z}, xmm2, xmm3/m16 {er}                                                                                                                                                                                                                                                                        | A   | V/V     | AVX512-FP16 | Multiply FP16 values from xmm1 and xmm2, subtract xmm3/m16, and store the result in xmm1 subject to writemask k1.                                       |
| EVEX.LLIG.66.MAP6.W0 BB /r VFMSUB231SH xmm1{k1}{z}, xmm2, xmm3/m16 {er}                                                                                                                                                                                                                                                                        | A   | V/V     | AVX512-FP16 | Multiply FP16 values from xmm2 and xmm3/m16, subtract xmm1, and store the result in xmm1 subject to writemask k1.                                       |
| EVEX.LLIG.66.MAP6.W0 9F /r VFNMSUB132SH xmm1{k1}{z}, xmm2, xmm3/m16 {er}                                                                                                                                                                                                                                                                       | A   | V/V     | AVX512-FP16 | Multiply FP16 values from xmm1 and xmm3/m16, and negate the value. Subtract xmm2 from this value, and store the result in xmm1 subject to writemask k1. |
| EVEX.LLIG.66.MAP6.W0 AF /r VFNMSUB213SH xmm1{k1}{z}, xmm2, xmm3/m16 {er}                                                                                                                                                                                                                                                                       | A   | V/V     | AVX512-FP16 | Multiply FP16 values from xmm1 and xmm2, and negate the value. Subtract xmm3/m16 from this value, and store the result in xmm1 subject to writemask k1. |
| EVEX.LLIG.66.MAP6.W0 BF /r VFNMSUB231SH xmm1{k1}{z}, xmm2, xmm3/m16 {er}                                                                                                                                                                                                                                                                       | A   | V/V     | AVX512-FP16 | Multiply FP16 values from xmm2 and xmm3/m16, and negate the value. Subtract xmm1 from this value, and store the result in xmm1 subject to writemask k1. |

## Instruction Operand Encoding

| Op/En | Tuple  | Operand 1        | Operand 2    | Operand 3     | Operand 4 |
| ----- | ------ | ---------------- | ------------ | ------------- | --------- |
| A     | Scalar | ModRM:reg (r, w) | VEX.vvvv (r) | ModRM:r/m (r) | N/A       |

### Description

This instruction performs a scalar multiply-subtract or negated multiply-subtract computation on the low FP16 values using three source operands and writes the result in the destination operand. The destination operand is also the first source operand. The “N” (negated) forms of this instruction subtract the remaining operand from the negated infinite precision intermediate product. The notation’ “132”, “213” and “231” indicate the use of the operands in ±A \* B − C, where each digit corresponds to the operand number, with the destination being operand 1; see [Table 5-7](/x86/vfmsub132sh:vfnmsub132sh:vfmsub213sh:vfnmsub213sh:vfmsub231sh:vfnmsub231sh#tbl-5-7).

Bits 127:16 of the destination operand are preserved. Bits MAXVL-1:128 of the destination operand are zeroed. The low FP16 element of the destination is updated according to the writemask.

| Notation | Operands                 |
| -------- | ------------------------ |
| 132      | dest = ± dest\*src3-src2 |
| 231      | dest = ± src2\*src3-dest |
| 213      | dest = ± src2\*dest-src3 |

[Table 5-7](/x86/vfmsub132sh:vfnmsub132sh:vfmsub213sh:vfnmsub213sh:vfmsub231sh:vfnmsub231sh#tbl-5-7). VF[,N]MSUB[132,213,231]SH Notation for Operands

### Operation

#### VF[,N]MSUB132SH DEST, SRC2, SRC3 (EVEX encoded versions)

```
IF EVEX.b = 1 and SRC3 is a register:
    SET_RM(EVEX.RC)
ELSE
    SET_RM(MXCSR.RC)
IF k1[0] OR *no writemask*:
    IF *negative form*:
        DEST.fp16[0] := RoundFPControl(-DEST.fp16[0]*SRC3.fp16[0] - SRC2.fp16[0])
    ELSE:
        DEST.fp16[0] := RoundFPControl(DEST.fp16[0]*SRC3.fp16[0] - SRC2.fp16[0])
ELSE IF *zeroing*:
    DEST.fp16[0] := 0
// else DEST.fp16[0] remains unchanged
//DEST[127:16] remains unchanged
DEST[MAXVL-1:128] := 0

```

#### VF[,N]MSUB213SH DEST, SRC2, SRC3 (EVEX encoded versions)

```
IF EVEX.b = 1 and SRC3 is a register:
    SET_RM(EVEX.RC)
ELSE
    SET_RM(MXCSR.RC)
IF k1[0] OR *no writemask*:
    IF *negative form:
        DEST.fp16[0] := RoundFPControl(-SRC2.fp16[0]*DEST.fp16[0] - SRC3.fp16[0])
    ELSE:
        DEST.fp16[0] := RoundFPControl(SRC2.fp16[0]*DEST.fp16[0] - SRC3.fp16[0])
ELSE IF *zeroing*:
    DEST.fp16[0] := 0
// else DEST.fp16[0] remains unchanged
//DEST[127:16] remains unchanged
DEST[MAXVL-1:128] := 0

```

#### VF[,N]MSUB231SH DEST, SRC2, SRC3 (EVEX encoded versions)

```
IF EVEX.b = 1 and SRC3 is a register:
    SET_RM(EVEX.RC)
ELSE
    SET_RM(MXCSR.RC)
IF k1[0] OR *no writemask*:
    IF *negative form*:
        DEST.fp16[0] := RoundFPControl(-SRC2.fp16[0]*SRC3.fp16[0] - DEST.fp16[0])
    ELSE:
        DEST.fp16[0] := RoundFPControl(SRC2.fp16[0]*SRC3.fp16[0] - DEST.fp16[0])
ELSE IF *zeroing*:
    DEST.fp16[0] := 0
// else DEST.fp16[0] remains unchanged
//DEST[127:16] remains unchanged
DEST[MAXVL-1:128] := 0

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
VFMSUB132SH, VFMSUB213SH, and VFMSUB231SH: __m128h _mm_fmsub_round_sh (__m128h a, __m128h b, __m128h c, const int rounding);

```

```
__m128h _mm_mask_fmsub_round_sh (__m128h a, __mmask8 k, __m128h b, __m128h c, const int rounding);

```

```
__m128h _mm_mask3_fmsub_round_sh (__m128h a, __m128h b, __m128h c, __mmask8 k, const int rounding);

```

```
__m128h _mm_maskz_fmsub_round_sh (__mmask8 k, __m128h a, __m128h b, __m128h c, const int rounding);

```

```
__m128h _mm_fmsub_sh (__m128h a, __m128h b, __m128h c);

```

```
__m128h _mm_mask_fmsub_sh (__m128h a, __mmask8 k, __m128h b, __m128h c);

```

```
__m128h _mm_mask3_fmsub_sh (__m128h a, __m128h b, __m128h c, __mmask8 k);

```

```
__m128h _mm_maskz_fmsub_sh (__mmask8 k, __m128h a, __m128h b, __m128h c);

```

```
VFNMSUB132SH, VFNMSUB213SH, and VFNMSUB231SH: __m128h _mm_fnmsub_round_sh (__m128h a, __m128h b, __m128h c, const int rounding);

```

```
__m128h _mm_mask_fnmsub_round_sh (__m128h a, __mmask8 k, __m128h b, __m128h c, const int rounding);

```

```
__m128h _mm_mask3_fnmsub_round_sh (__m128h a, __m128h b, __m128h c, __mmask8 k, const int rounding);

```

```
__m128h _mm_maskz_fnmsub_round_sh (__mmask8 k, __m128h a, __m128h b, __m128h c, const int rounding);

```

```
__m128h _mm_fnmsub_sh (__m128h a, __m128h b, __m128h c);

```

```
__m128h _mm_mask_fnmsub_sh (__m128h a, __mmask8 k, __m128h b, __m128h c);

```

```
__m128h _mm_mask3_fnmsub_sh (__m128h a, __m128h b, __m128h c, __mmask8 k);

```

```
__m128h _mm_maskz_fnmsub_sh (__mmask8 k, __m128h a, __m128h b, __m128h c);

```

### SIMD Floating-Point Exceptions

Invalid, Underflow, Overflow, Precision, Denormal

### Other Exceptions

EVEX-encoded instructions, see Table 2-47, “Type E3 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
