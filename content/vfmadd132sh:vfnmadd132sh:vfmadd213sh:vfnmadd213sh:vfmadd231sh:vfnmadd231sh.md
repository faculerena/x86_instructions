# VFMADD132SH/VFNMADD132SH/VFMADD213SH/VFNMADD213SH/VFMADD231SH/VFNMADD231SH

**Fused Multiply**

| Instruction En Bit Mode Flag Support Instruction En Bit Mode Flag Support 64/32 CPUID Feature Instruction En Bit Mode Flag CPUID Feature Instruction En Bit Mode Flag Op/ 64/32 CPUID Feature Instruction En Bit Mode Flag 64/32 CPUID Feature Instruction En Bit Mode Flag CPUID Feature Instruction En Bit Mode Flag Op/ 64/32 CPUID Feature |     | Support |             | Description                                                                                                              |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- | ------- | ----------- | ------------------------------------------------------------------------------------------------------------------------ |
| EVEX.LLIG.66.MAP6.W0 99 /r VFMADD132SH xmm1{k1}{z}, xmm2, xmm3/m16 {er}                                                                                                                                                                                                                                                                        | A   | V/V     | AVX512-FP16 | Multiply FP16 values from xmm1 and xmm3/m16, add to xmm2, and store the result in xmm1.                                  |
| EVEX.LLIG.66.MAP6.W0 A9 /r VFMADD213SH xmm1{k1}{z}, xmm2, xmm3/m16 {er}                                                                                                                                                                                                                                                                        | A   | V/V     | AVX512-FP16 | Multiply FP16 values from xmm1 and xmm2, add to xmm3/m16, and store the result in xmm1.                                  |
| EVEX.LLIG.66.MAP6.W0 B9 /r VFMADD231SH xmm1{k1}{z}, xmm2, xmm3/m16 {er}                                                                                                                                                                                                                                                                        | A   | V/V     | AVX512-FP16 | Multiply FP16 values from xmm2 and xmm3/m16, add to xmm1, and store the result in xmm1.                                  |
| EVEX.LLIG.66.MAP6.W0 9D /r VFNMADD132SH xmm1{k1}{z}, xmm2, xmm3/m16 {er}                                                                                                                                                                                                                                                                       | A   | V/V     | AVX512-FP16 | Multiply FP16 values from xmm1 and xmm3/m16, and negate the value. Add this value to xmm2, and store the result in xmm1. |
| EVEX.LLIG.66.MAP6.W0 AD /r VFNMADD213SH xmm1{k1}{z}, xmm2, xmm3/m16 {er}                                                                                                                                                                                                                                                                       | A   | V/V     | AVX512-FP16 | Multiply FP16 values from xmm1 and xmm2, and negate the value. Add this value to xmm3/m16, and store the result in xmm1. |
| EVEX.LLIG.66.MAP6.W0 BD /r VFNMADD231SH xmm1{k1}{z}, xmm2, xmm3/m16 {er}                                                                                                                                                                                                                                                                       | A   | V/V     | AVX512-FP16 | Multiply FP16 values from xmm2 and xmm3/m16, and negate the value. Add this value to xmm1, and store the result in xmm1. |

## Instruction Operand Encoding

| Op/En | Tuple  | Operand 1        | Operand 2    | Operand 3     | Operand 4 |
| ----- | ------ | ---------------- | ------------ | ------------- | --------- |
| A     | Scalar | ModRM:reg (r, w) | VEX.vvvv (r) | ModRM:r/m (r) | N/A       |

### Description

Performs a scalar multiply-add or negated multiply-add computation on the low FP16 values using three source operands and writes the result in the destination operand. The destination operand is also the first source operand. The “N” (negated) forms of this instruction add the negated infinite precision intermediate product to the corresponding remaining operand. The notation’ “132”, “213” and “231” indicate the use of the operands in ±A \* B + C, where each digit corresponds to the operand number, with the destination being operand 1; see [Table 5-4](/x86/vfmadd132sh:vfnmadd132sh:vfmadd213sh:vfnmadd213sh:vfmadd231sh:vfnmadd231sh#tbl-5-4).

Bits 127:16 of the destination operand are preserved. Bits MAXVL-1:128 of the destination operand are zeroed. The low FP16 element of the destination is updated according to the writemask.

| Notation | Operands                 |
| -------- | ------------------------ |
| 132      | dest = ± dest\*src3+src2 |
| 231      | dest = ± src2\*src3+dest |
| 213      | dest = ± src2\*dest+src3 |

[Table 5-4](/x86/vfmadd132sh:vfnmadd132sh:vfmadd213sh:vfnmadd213sh:vfmadd231sh:vfnmadd231sh#tbl-5-4). VF[,N]MADD[132,213,231]SH Notation for Operands

### Operation

#### VF[,N]MADD132SH DEST, SRC2, SRC3 (EVEX encoded versions)

```
IF EVEX.b = 1 and SRC3 is a register:
    SET_RM(EVEX.RC)
ELSE
    SET_RM(MXCSR.RC)
IF k1[0] OR *no writemask*:
    IF *negative form*:
        DEST.fp16[0] := RoundFPControl(-DEST.fp16[0]*SRC3.fp16[0] + SRC2.fp16[0])
    ELSE:
        DEST.fp16[0] := RoundFPControl(DEST.fp16[0]*SRC3.fp16[0] + SRC2.fp16[0])
ELSE IF *zeroing*:
    DEST.fp16[0] := 0
// else DEST.fp16[0] remains unchanged
//DEST[127:16] remains unchanged
DEST[MAXVL-1:128] := 0

```

#### VF[,N]MADD213SH DEST, SRC2, SRC3 (EVEX encoded versions)

```
IF EVEX.b = 1 and SRC3 is a register:
    SET_RM(EVEX.RC)
ELSE
    SET_RM(MXCSR.RC)
IF k1[0] OR *no writemask*:
    IF *negative form:
        DEST.fp16[0] := RoundFPControl(-SRC2.fp16[0]*DEST.fp16[0] + SRC3.fp16[0])
    ELSE:
        DEST.fp16[0] := RoundFPControl(SRC2.fp16[0]*DEST.fp16[0] + SRC3.fp16[0])
ELSE IF *zeroing*:
    DEST.fp16[0] := 0
// else DEST.fp16[0] remains unchanged
//DEST[127:16] remains unchanged
DEST[MAXVL-1:128] := 0

```

#### VF[,N]MADD231SH DEST, SRC2, SRC3 (EVEX encoded versions)

```
IF EVEX.b = 1 and SRC3 is a register:
    SET_RM(EVEX.RC)
ELSE
    SET_RM(MXCSR.RC)
IF k1[0] OR *no writemask*:
    IF *negative form*:
        DEST.fp16[0] := RoundFPControl(-SRC2.fp16[0]*SRC3.fp16[0] + DEST.fp16[0])
    ELSE:
        DEST.fp16[0] := RoundFPControl(SRC2.fp16[0]*SRC3.fp16[0] + DEST.fp16[0])
ELSE IF *zeroing*:
    DEST.fp16[0] := 0
// else DEST.fp16[0] remains unchanged
//DEST[127:16] remains unchanged
DEST[MAXVL-1:128] := 0

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
VFMADD132SH, VFMADD213SH, and VFMADD231SH: __m128h _mm_fmadd_round_sh (__m128h a, __m128h b, __m128h c, const int rounding);

```

```
__m128h _mm_mask_fmadd_round_sh (__m128h a, __mmask8 k, __m128h b, __m128h c, const int rounding);

```

```
__m128h _mm_mask3_fmadd_round_sh (__m128h a, __m128h b, __m128h c, __mmask8 k, const int rounding);

```

```
__m128h _mm_maskz_fmadd_round_sh (__mmask8 k, __m128h a, __m128h b, __m128h c, const int rounding);

```

```
__m128h _mm_fmadd_sh (__m128h a, __m128h b, __m128h c);

```

```
__m128h _mm_mask_fmadd_sh (__m128h a, __mmask8 k, __m128h b, __m128h c);

```

```
__m128h _mm_mask3_fmadd_sh (__m128h a, __m128h b, __m128h c, __mmask8 k);

```

```
__m128h _mm_maskz_fmadd_sh (__mmask8 k, __m128h a, __m128h b, __m128h c);

```

```
VFNMADD132SH, VFNMADD213SH, and VFNMADD231SH: __m128h _mm_fnmadd_round_sh (__m128h a, __m128h b, __m128h c, const int rounding);

```

```
__m128h _mm_mask_fnmadd_round_sh (__m128h a, __mmask8 k, __m128h b, __m128h c, const int rounding);

```

```
__m128h _mm_mask3_fnmadd_round_sh (__m128h a, __m128h b, __m128h c, __mmask8 k, const int rounding);

```

```
__m128h _mm_maskz_fnmadd_round_sh (__mmask8 k, __m128h a, __m128h b, __m128h c, const int rounding);

```

```
__m128h _mm_fnmadd_sh (__m128h a, __m128h b, __m128h c);

```

```
__m128h _mm_mask_fnmadd_sh (__m128h a, __mmask8 k, __m128h b, __m128h c);

```

```
__m128h _mm_mask3_fnmadd_sh (__m128h a, __m128h b, __m128h c, __mmask8 k);

```

```
__m128h _mm_maskz_fnmadd_sh (__mmask8 k, __m128h a, __m128h b, __m128h c);

```

### SIMD Floating-Point Exceptions

Invalid, Underflow, Overflow, Precision, Denormal

### Other Exceptions

EVEX-encoded instructions, see Table 2-47, “Type E3 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
