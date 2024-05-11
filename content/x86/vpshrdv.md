# VPSHRDV**Concatenate and Variable Shift Packed Data Right Logical**

| Opcode/Instruction                                                      | Op/En | 64/32 bit Mode Support | CPUID Feature Flag    | Description                                                                                     |
| ----------------------------------------------------------------------- | ----- | ---------------------- | --------------------- | ----------------------------------------------------------------------------------------------- |
| EVEX.128.66.0F38.W1 72 /r VPSHRDVW xmm1{k1}{z}, xmm2, xmm3/m128         | A     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate xmm1 and xmm2, extract result shifted to the right by value in xmm3/m128 into xmm1. |
| EVEX.256.66.0F38.W1 72 /r VPSHRDVW ymm1{k1}{z}, ymm2, ymm3/m256         | A     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate ymm1 and ymm2, extract result shifted to the right by value in xmm3/m256 into ymm1. |
| EVEX.512.66.0F38.W1 72 /r VPSHRDVW zmm1{k1}{z}, zmm2, zmm3/m512         | A     | V/V                    | AVX512_VBMI2          | Concatenate zmm1 and zmm2, extract result shifted to the right by value in zmm3/m512 into zmm1. |
| EVEX.128.66.0F38.W0 73 /r VPSHRDVD xmm1{k1}{z}, xmm2, xmm3/m128/m32bcst | B     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate xmm1 and xmm2, extract result shifted to the right by value in xmm3/m128 into xmm1. |
| EVEX.256.66.0F38.W0 73 /r VPSHRDVD ymm1{k1}{z}, ymm2, ymm3/m256/m32bcst | B     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate ymm1 and ymm2, extract result shifted to the right by value in xmm3/m256 into ymm1. |
| EVEX.512.66.0F38.W0 73 /r VPSHRDVD zmm1{k1}{z}, zmm2, zmm3/m512/m32bcst | B     | V/V                    | AVX512_VBMI2          | Concatenate zmm1 and zmm2, extract result shifted to the right by value in zmm3/m512 into zmm1. |
| EVEX.128.66.0F38.W1 73 /r VPSHRDVQ xmm1{k1}{z}, xmm2, xmm3/m128/m64bcst | B     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate xmm1 and xmm2, extract result shifted to the right by value in xmm3/m128 into xmm1. |
| EVEX.256.66.0F38.W1 73 /r VPSHRDVQ ymm1{k1}{z}, ymm2, ymm3/m256/m64bcst | B     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate ymm1 and ymm2, extract result shifted to the right by value in xmm3/m256 into ymm1. |
| EVEX.512.66.0F38.W1 73 /r VPSHRDVQ zmm1{k1}{z}, zmm2, zmm3/m512/m64bcst | B     | V/V                    | AVX512_VBMI2          | Concatenate zmm1 and zmm2, extract result shifted to the right by value in zmm3/m512 into zmm1. |

## Instruction Operand Encoding

| Op/En | Tuple    | Operand 1        | Operand 2     | Operand 3     | Operand 4 |
| ----- | -------- | ---------------- | ------------- | ------------- | --------- |
| A     | Full Mem | ModRM:reg (r, w) | EVEX.vvvv (r) | ModRM:r/m (r) | N/A       |
| B     | Full     | ModRM:reg (r, w) | EVEX.vvvv (r) | ModRM:r/m (r) | N/A       |

### Description

Concatenate packed data, extract result shifted to the right by variable value.

This instruction supports memory fault suppression.

### Operation

#### VPSHRDVW DEST, SRC2, SRC3

```
(KL, VL) = (8, 128), (16, 256), (32, 512)
FOR j := 0 TO KL-1:
    IF MaskBit(j) OR *no writemask*:
        DEST.word[j] := concat(SRC2.word[j], DEST.word[j]) >> (SRC3.word[j] & 15)
    ELSE IF *zeroing*:
        DEST.word[j] := 0
    *ELSE DEST.word[j] remains unchanged*
DEST[MAX_VL-1:VL] := 0

```

#### VPSHRDVD DEST, SRC2, SRC3

```
(KL, VL) = (4, 128), (8, 256), (16, 512)
FOR j := 0 TO KL-1:
    IF SRC3 is broadcast memop:
        tsrc3 := SRC3.dword[0]
    ELSE:
        tsrc3 := SRC3.dword[j]
    IF MaskBit(j) OR *no writemask*:
        DEST.dword[j] := concat(SRC2.dword[j], DEST.dword[j]) >> (tsrc3 & 31)
    ELSE IF *zeroing*:
        DEST.dword[j] := 0
    *ELSE DEST.dword[j] remains unchanged*
DEST[MAX_VL-1:VL] := 0

```

#### VPSHRDVQ DEST, SRC2, SRC3

```
(KL, VL) = (2, 128), (4, 256), (8, 512)
FOR j := 0 TO KL-1:
    IF SRC3 is broadcast memop:
        tsrc3 := SRC3.qword[0]
    ELSE:
        tsrc3 := SRC3.qword[j]
    IF MaskBit(j) OR *no writemask*:
        DEST.qword[j] := concat(SRC2.qword[j], DEST.qword[j]) >> (tsrc3 & 63)
    ELSE IF *zeroing*:
        DEST.qword[j] := 0
    *ELSE DEST.qword[j] remains unchanged*
DEST[MAX_VL-1:VL] := 0

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
VPSHRDVQ __m128i _mm_shrdv_epi64(__m128i, __m128i, __m128i);

```

```
VPSHRDVQ __m128i _mm_mask_shrdv_epi64(__m128i, __mmask8, __m128i, __m128i);

```

```
VPSHRDVQ __m128i _mm_maskz_shrdv_epi64(__mmask8, __m128i, __m128i, __m128i);

```

```
VPSHRDVQ __m256i _mm256_shrdv_epi64(__m256i, __m256i, __m256i);

```

```
VPSHRDVQ __m256i _mm256_mask_shrdv_epi64(__m256i, __mmask8, __m256i, __m256i);

```

```
VPSHRDVQ __m256i _mm256_maskz_shrdv_epi64(__mmask8, __m256i, __m256i, __m256i);

```

```
VPSHRDVQ __m512i _mm512_shrdv_epi64(__m512i, __m512i, __m512i);

```

```
VPSHRDVQ __m512i _mm512_mask_shrdv_epi64(__m512i, __mmask8, __m512i, __m512i);

```

```
VPSHRDVQ __m512i _mm512_maskz_shrdv_epi64(__mmask8, __m512i, __m512i, __m512i);

```

```
VPSHRDVD __m128i _mm_shrdv_epi32(__m128i, __m128i, __m128i);

```

```
VPSHRDVD __m128i _mm_mask_shrdv_epi32(__m128i, __mmask8, __m128i, __m128i);

```

```
VPSHRDVD __m128i _mm_maskz_shrdv_epi32(__mmask8, __m128i, __m128i, __m128i);

```

```
VPSHRDVD __m256i _mm256_shrdv_epi32(__m256i, __m256i, __m256i);

```

```
VPSHRDVD __m256i _mm256_mask_shrdv_epi32(__m256i, __mmask8, __m256i, __m256i);

```

```
VPSHRDVD __m256i _mm256_maskz_shrdv_epi32(__mmask8, __m256i, __m256i, __m256i);

```

```
VPSHRDVD __m512i _mm512_shrdv_epi32(__m512i, __m512i, __m512i);

```

```
VPSHRDVD __m512i _mm512_mask_shrdv_epi32(__m512i, __mmask16, __m512i, __m512i);

```

```
VPSHRDVD __m512i _mm512_maskz_shrdv_epi32(__mmask16, __m512i, __m512i, __m512i);

```

```
VPSHRDVW __m128i _mm_shrdv_epi16(__m128i, __m128i, __m128i);

```

```
VPSHRDVW __m128i _mm_mask_shrdv_epi16(__m128i, __mmask8, __m128i, __m128i);

```

```
VPSHRDVW __m128i _mm_maskz_shrdv_epi16(__mmask8, __m128i, __m128i, __m128i);

```

```
VPSHRDVW __m256i _mm256_shrdv_epi16(__m256i, __m256i, __m256i);

```

```
VPSHRDVW __m256i _mm256_mask_shrdv_epi16(__m256i, __mmask16, __m256i, __m256i);

```

```
VPSHRDVW __m256i _mm256_maskz_shrdv_epi16(__mmask16, __m256i, __m256i, __m256i);

```

```
VPSHRDVW __m512i _mm512_shrdv_epi16(__m512i, __m512i, __m512i);

```

```
VPSHRDVW __m512i _mm512_mask_shrdv_epi16(__m512i, __mmask32, __m512i, __m512i);

```

```
VPSHRDVW __m512i _mm512_maskz_shrdv_epi16(__mmask32, __m512i, __m512i, __m512i);

```

### SIMD Floating-Point Exceptions

None.

### Other Exceptions

See Table 2-49, “Type E4 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
