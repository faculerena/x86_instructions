# VRSQRTPH**Compute Reciprocals of Square Roots of Packed FP**

| Instruction En bit Mode Flag Support Instruction En bit Mode Flag Support 64/32 CPUID Feature Instruction En bit Mode Flag CPUID Feature Instruction En bit Mode Flag Op/ 64/32 CPUID Feature Instruction En bit Mode Flag 64/32 CPUID Feature Instruction En bit Mode Flag CPUID Feature Instruction En bit Mode Flag Op/ 64/32 CPUID Feature |     | Support |                      | Description                                                                                                                                              |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- | ------- | -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| EVEX.128.66.MAP6.W0 4E /r VRSQRTPH xmm1{k1}{z}, xmm2/m128/m16bcst                                                                                                                                                                                                                                                                              | A   | V/V     | AVX512-FP16 AVX512VL | Compute the approximate reciprocals of the square roots of packed FP16 values in xmm2/m128/m16bcst and store the result in xmm1 subject to writemask k1. |
| EVEX.256.66.MAP6.W0 4E /r VRSQRTPH ymm1{k1}{z}, ymm2/m256/m16bcst                                                                                                                                                                                                                                                                              | A   | V/V     | AVX512-FP16 AVX512VL | Compute the approximate reciprocals of the square roots of packed FP16 values in ymm2/m256/m16bcst and store the result in ymm1 subject to writemask k1. |
| EVEX.512.66.MAP6.W0 4E /r VRSQRTPH zmm1{k1}{z}, zmm2/m512/m16bcst                                                                                                                                                                                                                                                                              | A   | V/V     | AVX512-FP16          | Compute the approximate reciprocals of the square roots of packed FP16 values in zmm2/m512/m16bcst and store the result in zmm1 subject to writemask k1. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1     | Operand 2     | Operand 3 | Operand 4 |
| ----- | ----- | ------------- | ------------- | --------- | --------- |
| A     | Full  | ModRM:reg (w) | ModRM:r/m (r) | N/A       | N/A       |

### Description

This instruction performs a SIMD computation of the approximate reciprocals square-root of 8/16/32 packed FP16 floating-point values in the source operand (the second operand) and stores the packed FP16 floating-point results in the destination operand.

The maximum relative error for this approximation is less than 2−11 + 2−14. For special cases, see [Table 5-38](/x86/vrsqrtph#tbl-5-38).

The destination elements are updated according to the writemask.

| Input value  | Reset Value     | Comments                 |
| ------------ | --------------- | ------------------------ |
| Any denormal | Normal          | Cannot generate overflow |
| X = 2*−2n*   | 2*n*            |                          |
| X<0          | QNaN_Indefinite | Including −∞             |
| X = −0       | −∞              |                          |
| X = +0       | +∞              |                          |
| X = +∞       | +0              |                          |

[Table 5-38](/x86/vrsqrtph#tbl-5-38). VRSQRTPH/VRSQRTSH Special Cases

### Operation

#### VRSQRTPH dest{k1}, src

```
VL = 128, 256 or 512
KL := VL/16
FOR i := 0 to KL-1:
    IF k1[i] or *no writemask*:
        IF SRC is memory and (EVEX.b = 1):
            tsrc := src.fp16[0]
        ELSE:
            tsrc := src.fp16[i]
        DEST.fp16[i] := APPROXIMATE(1.0 / SQRT(tsrc) )
    ELSE IF *zeroing*:
        DEST.fp16[i] := 0
    //else DEST.fp16[i] remains unchanged
DEST[MAXVL-1:VL] := 0

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
VRSQRTPH __m128h _mm_mask_rsqrt_ph (__m128h src, __mmask8 k, __m128h a);

```

```
VRSQRTPH __m128h _mm_maskz_rsqrt_ph (__mmask8 k, __m128h a);

```

```
VRSQRTPH __m128h _mm_rsqrt_ph (__m128h a);

```

```
VRSQRTPH __m256h _mm256_mask_rsqrt_ph (__m256h src, __mmask16 k, __m256h a);

```

```
VRSQRTPH __m256h _mm256_maskz_rsqrt_ph (__mmask16 k, __m256h a);

```

```
VRSQRTPH __m256h _mm256_rsqrt_ph (__m256h a);

```

```
VRSQRTPH __m512h _mm512_mask_rsqrt_ph (__m512h src, __mmask32 k, __m512h a);

```

```
VRSQRTPH __m512h _mm512_maskz_rsqrt_ph (__mmask32 k, __m512h a);

```

```
VRSQRTPH __m512h _mm512_rsqrt_ph (__m512h a);

```

### SIMD Floating-Point Exceptions

None.

### Other Exceptions

EVEX-encoded instruction, see Table 2-49, “Type E4 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
