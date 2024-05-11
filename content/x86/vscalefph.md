# VSCALEFPH

**Scale Packed FP**

| Instruction En bit Mode Flag Support Instruction En bit Mode Flag Support 64/32 CPUID Feature Instruction En bit Mode Flag CPUID Feature Instruction En bit Mode Flag Op/ 64/32 CPUID Feature Instruction En bit Mode Flag 64/32 CPUID Feature Instruction En bit Mode Flag CPUID Feature Instruction En bit Mode Flag Op/ 64/32 CPUID Feature |     | Support |                      | Description                                                                                                                     |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- | ------- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| EVEX.128.66.MAP6.W0 2C /r VSCALEFPH xmm1{k1}{z}, xmm2, xmm3/m128/m16bcst                                                                                                                                                                                                                                                                       | A   | V/V     | AVX512-FP16 AVX512VL | Scale the packed FP16 values in xmm2 using values from xmm3/m128/m16bcst, and store the result in xmm1 subject to writemask k1. |
| EVEX.256.66.MAP6.W0 2C /r VSCALEFPH ymm1{k1}{z}, ymm2, ymm3/m256/m16bcst                                                                                                                                                                                                                                                                       | A   | V/V     | AVX512-FP16 AVX512VL | Scale the packed FP16 values in ymm2 using values from ymm3/m256/m16bcst, and store the result in ymm1 subject to writemask k1. |
| EVEX.512.66.MAP6.W0 2C /r VSCALEFPH zmm1{k1}{z}, zmm2, zmm3/m512/m16bcst {er}                                                                                                                                                                                                                                                                  | A   | V/V     | AVX512-FP16          | Scale the packed FP16 values in zmm2 using values from zmm3/m512/m16bcst, and store the result in zmm1 subject to writemask k1. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1     | Operand 2    | Operand 3     | Operand 4 |
| ----- | ----- | ------------- | ------------ | ------------- | --------- |
| A     | Full  | ModRM:reg (w) | VEX.vvvv (r) | ModRM:r/m (r) | N/A       |

### Description

This instruction performs a floating-point scale of the packed FP16 values in the first source operand by multiplying it by 2 to the power of the FP16 values in second source operand. The destination elements are updated according to the writemask.

The equation of this operation is given by:

zmm1 := zmm2 \* 2floor(zmm3).

Floor(zmm3) means maximum integer value ≤ zmm3.

If the result cannot be represented in FP16, then the proper overflow response (for positive scaling operand), or the proper underflow response (for negative scaling operand), is issued. The overflow and underflow responses are dependent on the rounding mode (for IEEE-compliant rounding), as well as on other settings in MXCSR (exception mask bits), and on the SAE bit.

Handling of special-case input values are listed in [Table 5-41](/x86/vscalefph#tbl-5-41) and [Table 5-42](/x86/vscalefph#tbl-5-42).

| Src1        | Src2       |                  |                 |                | Set IE                   |
| ----------- | ---------- | ---------------- | --------------- | -------------- | ------------------------ |
| ±NaN        | +INF       | −INF             | 0/Denorm/Norm   |
| ±QNaN       | QNaN(Src1) | +INF             | +0              | QNaN(Src1)     | IF either source is SNaN |
| ±SNaN       | QNaN(Src1) | QNaN(Src1)       | QNaN(Src1)      | QNaN(Src1)     | YES                      |
| ±INF        | QNaN(Src2) | Src1             | QNaN_Indefinite | Src1           | IF Src2 is SNaN or −INF  |
| ±0          | QNaN(Src2) | QNaN_Indefinite  | Src1            | Src1           | IF Src2 is SNaN or +INF  |
| Denorm/Norm | QNaN(Src2) | ±INF (Src1 sign) | ±0 (Src1 sign)  | Compute Result | IF Src2 is SNaN          |

[Table 5-41](/x86/vscalefph#tbl-5-41). VSCALEFPH/VSCALEFSH Special Cases

| Special Case | Returned Value | Faults |
| ------------ | -------------- | ------ | --------------------------------------------- | --------- |
|              | result         | < 2-24 | ±0 or ±Min-Denormal (Src1 sign)               | Underflow |
|              | result         | ≥ 216  | ±INF (Src1 sign) or ±Max-Denormal (Src1 sign) | Overflow  |

[Table 5-42](/x86/vscalefph#tbl-5-42). Additional VSCALEFPH/VSCALEFSH Special Cases

### Operation

```
def scale_fp16(src1,src2):
    tmp1 := src1
    tmp2 := src2
    return tmp1 * POW(2, FLOOR(tmp2))

```

#### VSCALEFPH dest{k1}, src1, src2

```
VL = 128, 256, or 512
KL := VL / 16
IF (VL = 512) AND (EVEX.b = 1) and no memory operand:
    SET_RM(EVEX.RC)
ELSE
    SET_RM(MXCSR.RC)
FOR i := 0 to KL-1:
    IF k1[i] or *no writemask*:
        IF SRC2 is memory and (EVEX.b = 1):
            tsrc := src2.fp16[0]
        ELSE:
            tsrc := src2.fp16[i]
        dest.fp16[i] := scale_fp16(src1.fp16[i],tsrc)
    ELSE IF *zeroing*:
        dest.fp16[i] := 0
    //else dest.fp16[i] remains unchanged
DEST[MAXVL-1:VL] := 0

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
VSCALEFPH __m128h _mm_mask_scalef_ph (__m128h src, __mmask8 k, __m128h a, __m128h b);

```

```
VSCALEFPH __m128h _mm_maskz_scalef_ph (__mmask8 k, __m128h a, __m128h b);

```

```
VSCALEFPH __m128h _mm_scalef_ph (__m128h a, __m128h b);

```

```
VSCALEFPH __m256h _mm256_mask_scalef_ph (__m256h src, __mmask16 k, __m256h a, __m256h b);

```

```
VSCALEFPH __m256h _mm256_maskz_scalef_ph (__mmask16 k, __m256h a, __m256h b);

```

```
VSCALEFPH __m256h _mm256_scalef_ph (__m256h a, __m256h b);

```

```
VSCALEFPH __m512h _mm512_mask_scalef_ph (__m512h src, __mmask32 k, __m512h a, __m512h b);

```

```
VSCALEFPH __m512h _mm512_maskz_scalef_ph (__mmask32 k, __m512h a, __m512h b);

```

```
VSCALEFPH __m512h _mm512_scalef_ph (__m512h a, __m512h b);

```

```
VSCALEFPH __m512h _mm512_mask_scalef_round_ph (__m512h src, __mmask32 k, __m512h a, __m512h b, const int rounding);

```

```
VSCALEFPH __m512h _mm512_maskz_scalef_round_ph (__mmask32 k, __m512h a, __m512h b, const int;

```

```
VSCALEFPH __m512h _mm512_scalef_round_ph (__m512h a, __m512h b, const int rounding);

```

### SIMD Floating-Point Exceptions

Invalid, Underflow, Overflow, Precision, Denormal.

### Other Exceptions

EVEX-encoded instruction, see Table 2-46, “Type E2 Class Exception Conditions”.

Denormal-operand exception (#​D) is checked and signaled for src1 operand, but not for src2 operand. The denormal-operand exception is checked for src1 operand only if the src2 operand is not NaN. If the src2 operand is NaN, the processor generates NaN and does not signal denormal-operand exception, even if src1 operand is denormal.

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
