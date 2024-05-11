# POR**Bitwise Logical OR**

| Opcode/Instruction                                                  | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                                |
| ------------------------------------------------------------------- | ----- | ---------------------- | ------------------ | ------------------------------------------------------------------------------------------ |
| NP 0F EB /r1 POR mm, mm/m64                                         | A     | V/V                    | MMX                | Bitwise OR of mm/m64 and mm.                                                               |
| 66 0F EB /r POR xmm1, xmm2/m128                                     | A     | V/V                    | SSE2               | Bitwise OR of xmm2/m128 and xmm1.                                                          |
| VEX.128.66.0F.WIG EB /r VPOR xmm1, xmm2, xmm3/m128                  | B     | V/V                    | AVX                | Bitwise OR of xmm2/m128 and xmm3.                                                          |
| VEX.256.66.0F.WIG EB /r VPOR ymm1, ymm2, ymm3/m256                  | B     | V/V                    | AVX2               | Bitwise OR of ymm2/m256 and ymm3.                                                          |
| EVEX.128.66.0F.W0 EB /r VPORD xmm1 {k1}{z}, xmm2, xmm3/m128/m32bcst | C     | V/V                    | AVX512VL AVX512F   | Bitwise OR of packed doubleword integers in xmm2 and xmm3/m128/m32bcst using writemask k1. |
| EVEX.256.66.0F.W0 EB /r VPORD ymm1 {k1}{z}, ymm2, ymm3/m256/m32bcst | C     | V/V                    | AVX512VL AVX512F   | Bitwise OR of packed doubleword integers in ymm2 and ymm3/m256/m32bcst using writemask k1. |
| EVEX.512.66.0F.W0 EB /r VPORD zmm1 {k1}{z}, zmm2, zmm3/m512/m32bcst | C     | V/V                    | AVX512F            | Bitwise OR of packed doubleword integers in zmm2 and zmm3/m512/m32bcst using writemask k1. |
| EVEX.128.66.0F.W1 EB /r VPORQ xmm1 {k1}{z}, xmm2, xmm3/m128/m64bcst | C     | V/V                    | AVX512VL AVX512F   | Bitwise OR of packed quadword integers in xmm2 and xmm3/m128/m64bcst using writemask k1.   |
| EVEX.256.66.0F.W1 EB /r VPORQ ymm1 {k1}{z}, ymm2, ymm3/m256/m64bcst | C     | V/V                    | AVX512VL AVX512F   | Bitwise OR of packed quadword integers in ymm2 and ymm3/m256/m64bcst using writemask k1.   |
| EVEX.512.66.0F.W1 EB /r VPORQ zmm1 {k1}{z}, zmm2, zmm3/m512/m64bcst | C     | V/V                    | AVX512F            | Bitwise OR of packed quadword integers in zmm2 and zmm3/m512/m64bcst using writemask k1.   |

> 1. See note in Section 2.5, “Intel® AVX and Intel® SSE Instruction Exception Classification,” in the Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 2A, and Section 23.25.3, “Exception Conditions of Legacy SIMD Instructions Operating on MMX Registers,” in the Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 3B.

## Instruction Operand Encoding

| Op/En | Tuple Type | Operand 1        | Operand 2     | Operand 3     | Operand 4 |
| ----- | ---------- | ---------------- | ------------- | ------------- | --------- |
| A     | N/A        | ModRM:reg (r, w) | ModRM:r/m (r) | N/A           | N/A       |
| B     | N/A        | ModRM:reg (w)    | VEX.vvvv (r)  | ModRM:r/m (r) | N/A       |
| C     | Full       | ModRM:reg (w)    | EVEX.vvvv (r) | ModRM:r/m (r) | N/A       |

## Description

Performs a bitwise logical OR operation on the source operand (second operand) and the destination operand (first operand) and stores the result in the destination operand. Each bit of the result is set to 1 if either or both of the corresponding bits of the first and second operands are 1; otherwise, it is set to 0.

In 64-bit mode and not encoded with VEX/EVEX, using a REX prefix in the form of REX.R permits this instruction to access additional registers (XMM8-XMM15).

Legacy SSE version: The source operand can be an MMX technology register or a 64-bit memory location. The destination operand is an MMX technology register.

128-bit Legacy SSE version: The second source operand is an XMM register or a 128-bit memory location. The first source and destination operands can be XMM registers. Bits (MAXVL-1:128) of the corresponding YMM destination register remain unchanged.

VEX.128 encoded version: The second source operand is an XMM register or a 128-bit memory location. The first source and destination operands can be XMM registers. Bits (MAXVL-1:128) of the destination YMM register are zeroed.

VEX.256 encoded version: The second source operand is an YMM register or a 256-bit memory location. The first source and destination operands can be YMM registers.

EVEX encoded version: The first source operand is a ZMM/YMM/XMM register. The second source operand can be a ZMM/YMM/XMM register, a 512/256/128-bit memory location or a 512/256/128-bit vector broadcasted from a 32/64-bit memory location. The destination operand is a ZMM/YMM/XMM register conditionally updated with write-mask k1 at 32/64-bit granularity.

## Operation

### POR (64-bit Operand)

```
DEST := DEST OR SRC

```

### POR (128-bit Legacy SSE Version)

```
DEST := DEST OR SRC
DEST[MAXVL-1:128] (Unmodified)

```

### VPOR (VEX.128 Encoded Version)

```
DEST := SRC1 OR SRC2
DEST[MAXVL-1:128] := 0

```

### VPOR (VEX.256 Encoded Version)

```
DEST := SRC1 OR SRC2
DEST[MAXVL-1:256] := 0

```

### VPORD (EVEX Encoded Versions)

```
(KL, VL) = (4, 128), (8, 256), (16, 512)
FOR j := 0 TO KL-1
    i := j * 32
    IF k1[j] OR *no writemask* THEN
            IF (EVEX.b = 1) AND (SRC2 *is memory*)
                THEN DEST[i+31:i] := SRC1[i+31:i] BITWISE OR SRC2[31:0]
                ELSE DEST[i+31:i] := SRC1[i+31:i] BITWISE OR SRC2[i+31:i]
            FI;
        ELSE
            IF *merging-masking* ; merging-masking
                *DEST[i+31:i] remains unchanged*
                ELSE
                        ; zeroing-masking
                    DEST[i+31:i] := 0
            FI;
    FI;
ENDFOR;
DEST[MAXVL-1:VL] := 0

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
VPORD __m512i _mm512_or_epi32(__m512i a, __m512i b);

```

```
VPORD __m512i _mm512_mask_or_epi32(__m512i s, __mmask16 k, __m512i a, __m512i b);

```

```
VPORD __m512i _mm512_maskz_or_epi32( __mmask16 k, __m512i a, __m512i b);

```

```
VPORD __m256i _mm256_or_epi32(__m256i a, __m256i b);

```

```
VPORD __m256i _mm256_mask_or_epi32(__m256i s, __mmask8 k, __m256i a, __m256i b,);

```

```
VPORD __m256i _mm256_maskz_or_epi32( __mmask8 k, __m256i a, __m256i b);

```

```
VPORD __m128i _mm_or_epi32(__m128i a, __m128i b);

```

```
VPORD __m128i _mm_mask_or_epi32(__m128i s, __mmask8 k, __m128i a, __m128i b);

```

```
VPORD __m128i _mm_maskz_or_epi32( __mmask8 k, __m128i a, __m128i b);

```

```
VPORQ __m512i _mm512_or_epi64(__m512i a, __m512i b);

```

```
VPORQ __m512i _mm512_mask_or_epi64(__m512i s, __mmask8 k, __m512i a, __m512i b);

```

```
VPORQ __m512i _mm512_maskz_or_epi64(__mmask8 k, __m512i a, __m512i b);

```

```
VPORQ __m256i _mm256_or_epi64(__m256i a, int imm);

```

```
VPORQ __m256i _mm256_mask_or_epi64(__m256i s, __mmask8 k, __m256i a, __m256i b);

```

```
VPORQ __m256i _mm256_maskz_or_epi64( __mmask8 k, __m256i a, __m256i b);

```

```
VPORQ __m128i _mm_or_epi64(__m128i a, __m128i b);

```

```
VPORQ __m128i _mm_mask_or_epi64(__m128i s, __mmask8 k, __m128i a, __m128i b);

```

```
VPORQ __m128i _mm_maskz_or_epi64( __mmask8 k, __m128i a, __m128i b);

```

```
POR __m64 _mm_or_si64(__m64 m1, __m64 m2)

```

```
(V)POR __m128i _mm_or_si128(__m128i m1, __m128i m2)

```

```
VPOR __m256i _mm256_or_si256 ( __m256i a, __m256i b)

```

## Flags Affected

None.

## SIMD Floating-Point Exceptions

None.

## Other Exceptions

Non-EVEX-encoded instruction, see Table 2-21, “Type 4 Class Exception Conditions.”

EVEX-encoded instruction, see Table 2-49, “Type E4 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
