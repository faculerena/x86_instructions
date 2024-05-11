# VPOPCNT

**Return the Count of Number of Bits Set to **

| Opcode/Instruction                                                | Op/En | 64/32 bit Mode Support | CPUID Feature Flag        | Description                                                                                              |
| ----------------------------------------------------------------- | ----- | ---------------------- | ------------------------- | -------------------------------------------------------------------------------------------------------- |
| EVEX.128.66.0F38.W0 54 /r VPOPCNTB xmm1{k1}{z}, xmm2/m128         | A     | V/V                    | AVX512_BITALG AVX512VL    | Counts the number of bits set to one in xmm2/m128 and puts the result in xmm1 with writemask k1.         |
| EVEX.256.66.0F38.W0 54 /r VPOPCNTB ymm1{k1}{z}, ymm2/m256         | A     | V/V                    | AVX512_BITALG AVX512VL    | Counts the number of bits set to one in ymm2/m256 and puts the result in ymm1 with writemask k1.         |
| EVEX.512.66.0F38.W0 54 /r VPOPCNTB zmm1{k1}{z}, zmm2/m512         | A     | V/V                    | AVX512_BITALG             | Counts the number of bits set to one in zmm2/m512 and puts the result in zmm1 with writemask k1.         |
| EVEX.128.66.0F38.W1 54 /r VPOPCNTW xmm1{k1}{z}, xmm2/m128         | A     | V/V                    | AVX512_BITALG AVX512VL    | Counts the number of bits set to one in xmm2/m128 and puts the result in xmm1 with writemask k1.         |
| EVEX.256.66.0F38.W1 54 /r VPOPCNTW ymm1{k1}{z}, ymm2/m256         | A     | V/V                    | AVX512_BITALG AVX512VL    | Counts the number of bits set to one in ymm2/m256 and puts the result in ymm1 with writemask k1.         |
| EVEX.512.66.0F38.W1 54 /r VPOPCNTW zmm1{k1}{z}, zmm2/m512         | A     | V/V                    | AVX512_BITALG             | Counts the number of bits set to one in zmm2/m512 and puts the result in zmm1 with writemask k1.         |
| EVEX.128.66.0F38.W0 55 /r VPOPCNTD xmm1{k1}{z}, xmm2/m128/m32bcst | B     | V/V                    | AVX512_VPOPCNTDQ AVX512VL | Counts the number of bits set to one in xmm2/m128/m32bcst and puts the result in xmm1 with writemask k1. |
| EVEX.256.66.0F38.W0 55 /r VPOPCNTD ymm1{k1}{z}, ymm2/m256/m32bcst | B     | V/V                    | AVX512_VPOPCNTDQ AVX512VL | Counts the number of bits set to one in ymm2/m256/m32bcst and puts the result in ymm1 with writemask k1. |
| EVEX.512.66.0F38.W0 55 /r VPOPCNTD zmm1{k1}{z}, zmm2/m512/m32bcst | B     | V/V                    | AVX512_VPOPCNTDQ          | Counts the number of bits set to one in zmm2/m512/m32bcst and puts the result in zmm1 with writemask k1. |
| EVEX.128.66.0F38.W1 55 /r VPOPCNTQ xmm1{k1}{z}, xmm2/m128/m64bcst | B     | V/V                    | AVX512_VPOPCNTDQ AVX512VL | Counts the number of bits set to one in xmm2/m128/m32bcst and puts the result in xmm1 with writemask k1. |
| EVEX.256.66.0F38.W1 55 /r VPOPCNTQ ymm1{k1}{z}, ymm2/m256/m64bcst | B     | V/V                    | AVX512_VPOPCNTDQ AVX512VL | Counts the number of bits set to one in ymm2/m256/m32bcst and puts the result in ymm1 with writemask k1. |
| EVEX.512.66.0F38.W1 55 /r VPOPCNTQ zmm1{k1}{z}, zmm2/m512/m64bcst | B     | V/V                    | AVX512_VPOPCNTDQ          | Counts the number of bits set to one in zmm2/m512/m64bcst and puts the result in zmm1 with writemask k1. |

## Instruction Operand Encoding

| Op/En | Tuple    | Operand 1     | Operand 2     | Operand 3 | Operand 4 |
| ----- | -------- | ------------- | ------------- | --------- | --------- |
| A     | Full Mem | ModRM:reg (w) | ModRM:r/m (r) | N/A       | N/A       |
| B     | Full     | ModRM:reg (w) | ModRM:r/m (r) | N/A       | N/A       |

### Description

This instruction counts the number of bits set to one in each byte, word, dword or qword element of its source (e.g., zmm2 or memory) and places the results in the destination register (zmm1). This instruction supports memory fault suppression.

### Operation

#### VPOPCNTB

```
(KL, VL) = (16, 128), (32, 256), (64, 512)
FOR j := 0 TO KL-1:
    IF MaskBit(j) OR *no writemask*:
        DEST.byte[j] := POPCNT(SRC.byte[j])
    ELSE IF *merging-masking*:
        *DEST.byte[j] remains unchanged*
    ELSE:
        DEST.byte[j] := 0
DEST[MAX_VL-1:VL] := 0

```

#### VPOPCNTW

```
(KL, VL) = (8, 128), (16, 256), (32, 512)
FOR j := 0 TO KL-1:
    IF MaskBit(j) OR *no writemask*:
        DEST.word[j] := POPCNT(SRC.word[j])
    ELSE IF *merging-masking*:
        *DEST.word[j] remains unchanged*
    ELSE:
        DEST.word[j] := 0
DEST[MAX_VL-1:VL] := 0

```

#### VPOPCNTD

```
(KL, VL) = (4, 128), (8, 256), (16, 512)
FOR j := 0 TO KL-1:
    IF MaskBit(j) OR *no writemask*:
        IF SRC is broadcast memop:
            t := SRC.dword[0]
        ELSE:
            t := SRC.dword[j]
        DEST.dword[j] := POPCNT(t)
    ELSE IF *merging-masking*:
        *DEST..dword[j] remains unchanged*
    ELSE:
        DEST..dword[j] := 0
DEST[MAX_VL-1:VL] := 0

```

#### VPOPCNTQ

```
(KL, VL) = (2, 128), (4, 256), (8, 512)
FOR j := 0 TO KL-1:
    IF MaskBit(j) OR *no writemask*:
        IF SRC is broadcast memop:
            t := SRC.qword[0]
        ELSE:
            t := SRC.qword[j]
        DEST.qword[j] := POPCNT(t)
    ELSE IF *merging-masking*:
        *DEST..qword[j] remains unchanged*
    ELSE:
        DEST..qword[j] := 0
DEST[MAX_VL-1:VL] := 0

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
VPOPCNTW __m128i _mm_popcnt_epi16(__m128i);

```

```
VPOPCNTW __m128i _mm_mask_popcnt_epi16(__m128i, __mmask8, __m128i);

```

```
VPOPCNTW __m128i _mm_maskz_popcnt_epi16(__mmask8, __m128i);

```

```
VPOPCNTW __m256i _mm256_popcnt_epi16(__m256i);

```

```
VPOPCNTW __m256i _mm256_mask_popcnt_epi16(__m256i, __mmask16, __m256i);

```

```
VPOPCNTW __m256i _mm256_maskz_popcnt_epi16(__mmask16, __m256i);

```

```
VPOPCNTW __m512i _mm512_popcnt_epi16(__m512i);

```

```
VPOPCNTW __m512i _mm512_mask_popcnt_epi16(__m512i, __mmask32, __m512i);

```

```
VPOPCNTW __m512i _mm512_maskz_popcnt_epi16(__mmask32, __m512i);

```

```
VPOPCNTQ __m128i _mm_popcnt_epi64(__m128i);

```

```
VPOPCNTQ __m128i _mm_mask_popcnt_epi64(__m128i, __mmask8, __m128i);

```

```
VPOPCNTQ __m128i _mm_maskz_popcnt_epi64(__mmask8, __m128i);

```

```
VPOPCNTQ __m256i _mm256_popcnt_epi64(__m256i);

```

```
VPOPCNTQ __m256i _mm256_mask_popcnt_epi64(__m256i, __mmask8, __m256i);

```

```
VPOPCNTQ __m256i _mm256_maskz_popcnt_epi64(__mmask8, __m256i);

```

```
VPOPCNTQ __m512i _mm512_popcnt_epi64(__m512i);

```

```
VPOPCNTQ __m512i _mm512_mask_popcnt_epi64(__m512i, __mmask8, __m512i);

```

```
VPOPCNTQ __m512i _mm512_maskz_popcnt_epi64(__mmask8, __m512i);

```

```
VPOPCNTD __m128i _mm_popcnt_epi32(__m128i);

```

```
VPOPCNTD __m128i _mm_mask_popcnt_epi32(__m128i, __mmask8, __m128i);

```

```
VPOPCNTD __m128i _mm_maskz_popcnt_epi32(__mmask8, __m128i);

```

```
VPOPCNTD __m256i _mm256_popcnt_epi32(__m256i);

```

```
VPOPCNTD __m256i _mm256_mask_popcnt_epi32(__m256i, __mmask8, __m256i);

```

```
VPOPCNTD __m256i _mm256_maskz_popcnt_epi32(__mmask8, __m256i);

```

```
VPOPCNTD __m512i _mm512_popcnt_epi32(__m512i);

```

```
VPOPCNTD __m512i _mm512_mask_popcnt_epi32(__m512i, __mmask16, __m512i);

```

```
VPOPCNTD __m512i _mm512_maskz_popcnt_epi32(__mmask16, __m512i);

```

```
VPOPCNTB __m128i _mm_popcnt_epi8(__m128i);

```

```
VPOPCNTB __m128i _mm_mask_popcnt_epi8(__m128i, __mmask16, __m128i);

```

```
VPOPCNTB __m128i _mm_maskz_popcnt_epi8(__mmask16, __m128i);

```

```
VPOPCNTB __m256i _mm256_popcnt_epi8(__m256i);

```

```
VPOPCNTB __m256i _mm256_mask_popcnt_epi8(__m256i, __mmask32, __m256i);

```

```
VPOPCNTB __m256i _mm256_maskz_popcnt_epi8(__mmask32, __m256i);

```

```
VPOPCNTB __m512i _mm512_popcnt_epi8(__m512i);

```

```
VPOPCNTB __m512i _mm512_mask_popcnt_epi8(__m512i, __mmask64, __m512i);

```

```
VPOPCNTB __m512i _mm512_maskz_popcnt_epi8(__mmask64, __m512i);

```

### SIMD Floating-Point Exceptions

None.

### Other Exceptions

See Table 2-49, “Type E4 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
