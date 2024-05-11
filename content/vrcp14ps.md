# VRCP14PS

**Compute Approximate Reciprocals of Packed Float**

| Opcode/Instruction                                                 | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                                                                                                     |
| ------------------------------------------------------------------ | ----- | ---------------------- | ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| EVEX.128.66.0F38.W0 4C /r VRCP14PS xmm1 {k1}{z}, xmm2/m128/m32bcst | A     | V/V                    | AVX512VL AVX512F   | Computes the approximate reciprocals of the packed single-precision floating-point values in xmm2/m128/m32bcst and stores the results in xmm1. Under writemask. |
| EVEX.256.66.0F38.W0 4C /r VRCP14PS ymm1 {k1}{z}, ymm2/m256/m32bcst | A     | V/V                    | AVX512VL AVX512F   | Computes the approximate reciprocals of the packed single-precision floating-point values in ymm2/m256/m32bcst and stores the results in ymm1. Under writemask. |
| EVEX.512.66.0F38.W0 4C /r VRCP14PS zmm1 {k1}{z}, zmm2/m512/m32bcst | A     | V/V                    | AVX512F            | Computes the approximate reciprocals of the packed single-precision floating-point values in zmm2/m512/m32bcst and stores the results in zmm1. Under writemask. |

## Instruction Operand Encoding

| Op/En | Tuple Type | Operand 1     | Operand 2     | Operand 3 | Operand 4 |
| ----- | ---------- | ------------- | ------------- | --------- | --------- |
| A     | Full       | ModRM:reg (w) | ModRM:r/m (r) | N/A       | N/A       |

### Description

This instruction performs a SIMD computation of the approximate reciprocals of the packed single-precision floating-point values in the source operand (the second operand) and stores the packed single-precision floating-point results in the destination operand (the first operand). The maximum relative error for this approximation is less than 2-14.

The source operand can be a ZMM register, a 512-bit memory location or a 512-bit vector broadcasted from a 32-bit memory location. The destination operand is a ZMM register conditionally updated according to the writemask.

The VRCP14PS instruction is not affected by the rounding control bits in the MXCSR register. When a source value is a 0.0, an ∞ with the sign of the source value is returned. A denormal source value will be treated as zero only in case of DAZ bit set in MXCSR. Otherwise it is treated correctly (i.e., not as a 0.0). Underflow results are flushed to zero only in case of FTZ bit set in MXCSR. Otherwise it will be treated correctly (i.e., correct underflow result is written) with the sign of the operand. When a source value is a SNaN or QNaN, the SNaN is converted to a QNaN or the source QNaN is returned.

EVEX.vvvv is reserved and must be 1111b otherwise instructions will #​​​UD.

MXCSR exception flags are not affected by this instruction and floating-point exceptions are not reported.

| Input value     | Result value | Comments                                 |
| --------------- | ------------ | ---------------------------------------- |
| 0 ≤ X ≤ 2-128   | INF          | Very small denormal                      |
| -2-128 ≤ X ≤ -0 | -INF         | Very small denormal                      |
| X > 2126        | Underflow    | Up to 18 bits of fractions are returned1 |
| X < -2126       | -Underflow   | Up to 18 bits of fractions are returned1 |
| X = 2-n         | 2n           |                                          |
| X = -2-n        | -2n          |                                          |

[Table 5-27](/x86/vrcp14ps#tbl-5-27). VRCP14PS/VRCP14SS Special Cases

> 1. In this case, the mantissa is shifted right by one or two bits.

A numerically exact implementation of VRCP14xx can be found at:

### https://software.intel.com/en-us/articles/reference-implementations-for-IA-approximation-instructions-vrcp14-

### vrsqrt14-vrcp28-vrsqrt28-vexp2.

### Operation

#### VRCP14PS (EVEX encoded versions)

```
(KL, VL) = (4, 128), (8, 256), (16, 512)
FOR j := 0 TO KL-1
    i := j * 32
    IF k1[j] OR *no writemask* THEN
            IF (EVEX.b = 1) AND (SRC *is memory*)
                THEN DEST[i+31:i] := APPROXIMATE(1.0/SRC[31:0]);
                ELSE DEST[i+31:i] := APPROXIMATE(1.0/SRC[i+31:i]);
            FI;
    ELSE
        IF *merging-masking* ; merging-masking
            THEN *DEST[i+31:i] remains unchanged*
            ELSE ; zeroing-masking
                DEST[i+31:i] := 0
        FI;
    FI;
ENDFOR;
DEST[MAXVL-1:VL] := 0

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
VRCP14PS __m512 _mm512_rcp14_ps( __m512 a);

```

```
VRCP14PS __m512 _mm512_mask_rcp14_ps(__m512 s, __mmask16 k, __m512 a);

```

```
VRCP14PS __m512 _mm512_maskz_rcp14_ps( __mmask16 k, __m512 a);

```

```
VRCP14PS __m256 _mm256_rcp14_ps( __m256 a);

```

```
VRCP14PS __m256 _mm512_mask_rcp14_ps(__m256 s, __mmask8 k, __m256 a);

```

```
VRCP14PS __m256 _mm512_maskz_rcp14_ps( __mmask8 k, __m256 a);

```

```
VRCP14PS __m128 _mm_rcp14_ps( __m128 a);

```

```
VRCP14PS __m128 _mm_mask_rcp14_ps(__m128 s, __mmask8 k, __m128 a);

```

```
VRCP14PS __m128 _mm_maskz_rcp14_ps( __mmask8 k, __m128 a);

```

### SIMD Floating-Point Exceptions

None.

### Other Exceptions

See Table 2-49, “Type E4 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
