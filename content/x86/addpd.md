# ADDPD**Add Packed Double Precision Floating**

| Opcode/Instruction                                                       | Op / En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                                                                  |
| ------------------------------------------------------------------------ | ------- | ---------------------- | ------------------ | ---------------------------------------------------------------------------------------------------------------------------- |
| 66 0F 58 /r ADDPD xmm1, xmm2/m128                                        | A       | V/V                    | SSE2               | Add packed double precision floating-point values from xmm2/mem to xmm1 and store result in xmm1.                            |
| VEX.128.66.0F.WIG 58 /r VADDPD xmm1,xmm2, xmm3/m128                      | B       | V/V                    | AVX                | Add packed double precision floating-point values from xmm3/mem to xmm2 and store result in xmm1.                            |
| VEX.256.66.0F.WIG 58 /r VADDPD ymm1, ymm2, ymm3/m256                     | B       | V/V                    | AVX                | Add packed double precision floating-point values from ymm3/mem to ymm2 and store result in ymm1.                            |
| EVEX.128.66.0F.W1 58 /r VADDPD xmm1 {k1}{z}, xmm2, xmm3/m128/m64bcst     | C       | V/V                    | AVX512VL AVX512F   | Add packed double precision floating-point values from xmm3/m128/m64bcst to xmm2 and store result in xmm1 with writemask k1. |
| EVEX.256.66.0F.W1 58 /r VADDPD ymm1 {k1}{z}, ymm2, ymm3/m256/m64bcst     | C       | V/V                    | AVX512VL AVX512F   | Add packed double precision floating-point values from ymm3/m256/m64bcst to ymm2 and store result in ymm1 with writemask k1. |
| EVEX.512.66.0F.W1 58 /r VADDPD zmm1 {k1}{z}, zmm2, zmm3/m512/m64bcst{er} | C       | V/V                    | AVX512F            | Add packed double precision floating-point values from zmm3/m512/m64bcst to zmm2 and store result in zmm1 with writemask k1. |

## Instruction Operand Encoding

| Op/En | Tuple Type | Operand 1        | Operand 2     | Operand 3     | Operand 4 |
| ----- | ---------- | ---------------- | ------------- | ------------- | --------- |
| A     | N/A        | ModRM:reg (r, w) | ModRM:r/m (r) | N/A           | N/A       |
| B     | N/A        | ModRM:reg (w)    | VEX.vvvv (r)  | ModRM:r/m (r) | N/A       |
| C     | Full       | ModRM:reg (w)    | EVEX.vvvv (r) | ModRM:r/m (r) | N/A       |

## Description

Adds two, four or eight packed double precision floating-point values from the first source operand to the second source operand, and stores the packed double precision floating-point result in the destination operand.

EVEX encoded versions: The first source operand is a ZMM/YMM/XMM register. The second source operand can be a ZMM/YMM/XMM register, a 512/256/128-bit memory location or a 512/256/128-bit vector broadcasted from a 64-bit memory location. The destination operand is a ZMM/YMM/XMM register conditionally updated with writemask k1.

VEX.256 encoded version: The first source operand is a YMM register. The second source operand can be a YMM register or a 256-bit memory location. The destination operand is a YMM register. The upper bits (MAXVL-1:256) of the corresponding ZMM register destination are zeroed.

VEX.128 encoded version: the first source operand is a XMM register. The second source operand is an XMM register or 128-bit memory location. The destination operand is an XMM register. The upper bits (MAXVL-1:128) of the corresponding ZMM register destination are zeroed.

128-bit Legacy SSE version: The second source can be an XMM register or an 128-bit memory location. The destination is not distinct from the first source XMM register and the upper Bits (MAXVL-1:128) of the corresponding ZMM register destination are unmodified.

## Operation

### VADDPD (EVEX Encoded Versions) When SRC2 Operand is a Vector Register

```
(KL, VL) = (2, 128), (4, 256), (8, 512)
IF (VL = 512) AND (EVEX.b = 1)
    THEN
        SET_ROUNDING_MODE_FOR_THIS_INSTRUCTION(EVEX.RC);
    ELSE
        SET_ROUNDING_MODE_FOR_THIS_INSTRUCTION(MXCSR.RC);
FI;
FOR j := 0 TO KL-1
    i := j * 64
    IF k1[j] OR *no writemask*
        THEN DEST[i+63:i] := SRC1[i+63:i] + SRC2[i+63:i]
        ELSE
            IF *merging-masking* ; merging-masking
                THEN *DEST[i+63:i] remains unchanged*
                ELSE ; zeroing-masking
                    DEST[i+63:i] := 0
            FI
    FI;
ENDFOR
DEST[MAXVL-1:VL] := 0

```

### VADDPD (EVEX Encoded Versions) When SRC2 Operand is a Memory Source

```
(KL, VL) = (2, 128), (4, 256), (8, 512)
FOR j := 0 TO KL-1
    i := j * 64
    IF k1[j] OR *no writemask*
        THEN
            IF (EVEX.b = 1)
                THEN
                    DEST[i+63:i] := SRC1[i+63:i] + SRC2[63:0]
                ELSE
                    DEST[i+63:i] := SRC1[i+63:i] + SRC2[i+63:i]
            FI;
        ELSE
            IF *merging-masking* ; merging-masking
                THEN *DEST[i+63:i] remains unchanged*
                ELSE ; zeroing-masking
                    DEST[i+63:i] := 0
            FI
    FI;
ENDFOR
DEST[MAXVL-1:VL] := 0

```

### VADDPD (VEX.256 Encoded Version)

```
DEST[63:0] := SRC1[63:0] + SRC2[63:0]
DEST[127:64] := SRC1[127:64] + SRC2[127:64]
DEST[191:128] := SRC1[191:128] + SRC2[191:128]
DEST[255:192] := SRC1[255:192] + SRC2[255:192]
DEST[MAXVL-1:256] := 0
.

```

### VADDPD (VEX.128 Encoded Version)

```
DEST[63:0] := SRC1[63:0] + SRC2[63:0]
DEST[127:64] := SRC1[127:64] + SRC2[127:64]
DEST[MAXVL-1:128] := 0

```

### ADDPD (128-bit Legacy SSE Version)

```
DEST[63:0] := DEST[63:0] + SRC[63:0]
DEST[127:64] := DEST[127:64] + SRC[127:64]
DEST[MAXVL-1:128] (Unmodified)

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
VADDPD __m512d _mm512_add_pd (__m512d a, __m512d b);

```

```
VADDPD __m512d _mm512_mask_add_pd (__m512d s, __mmask8 k, __m512d a, __m512d b);

```

```
VADDPD __m512d _mm512_maskz_add_pd (__mmask8 k, __m512d a, __m512d b);

```

```
VADDPD __m256d _mm256_mask_add_pd (__m256d s, __mmask8 k, __m256d a, __m256d b);

```

```
VADDPD __m256d _mm256_maskz_add_pd (__mmask8 k, __m256d a, __m256d b);

```

```
VADDPD __m128d _mm_mask_add_pd (__m128d s, __mmask8 k, __m128d a, __m128d b);

```

```
VADDPD __m128d _mm_maskz_add_pd (__mmask8 k, __m128d a, __m128d b);

```

```
VADDPD __m512d _mm512_add_round_pd (__m512d a, __m512d b, int);

```

```
VADDPD __m512d _mm512_mask_add_round_pd (__m512d s, __mmask8 k, __m512d a, __m512d b, int);

```

```
VADDPD __m512d _mm512_maskz_add_round_pd (__mmask8 k, __m512d a, __m512d b, int);

```

```
ADDPD __m256d _mm256_add_pd (__m256d a, __m256d b);

```

```
ADDPD __m128d _mm_add_pd (__m128d a, __m128d b);

```

## SIMD Floating-Point Exceptions

Overflow, Underflow, Invalid, Precision, Denormal.

## Other Exceptions

VEX-encoded instruction, see Table 2-19, “Type 2 Class Exception Conditions.”

EVEX-encoded instruction, see Table 2-46, “Type E2 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
