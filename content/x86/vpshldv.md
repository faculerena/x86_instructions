# VPSHLDV**Concatenate and Variable Shift Packed Data Left Logical**

| Opcode/Instruction                                                      | Op/En | 64/32 bit Mode Support | CPUID Feature Flag    | Description                                                                                    |
| ----------------------------------------------------------------------- | ----- | ---------------------- | --------------------- | ---------------------------------------------------------------------------------------------- |
| EVEX.128.66.0F38.W1 70 /r VPSHLDVW xmm1{k1}{z}, xmm2, xmm3/m128         | A     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate xmm1 and xmm2, extract result shifted to the left by value in xmm3/m128 into xmm1. |
| EVEX.256.66.0F38.W1 70 /r VPSHLDVW ymm1{k1}{z}, ymm2, ymm3/m256         | A     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate ymm1 and ymm2, extract result shifted to the left by value in xmm3/m256 into ymm1. |
| EVEX.512.66.0F38.W1 70 /r VPSHLDVW zmm1{k1}{z}, zmm2, zmm3/m512         | A     | V/V                    | AVX512_VBMI2          | Concatenate zmm1 and zmm2, extract result shifted to the left by value in zmm3/m512 into zmm1. |
| EVEX.128.66.0F38.W0 71 /r VPSHLDVD xmm1{k1}{z}, xmm2, xmm3/m128/m32bcst | B     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate xmm1 and xmm2, extract result shifted to the left by value in xmm3/m128 into xmm1. |
| EVEX.256.66.0F38.W0 71 /r VPSHLDVD ymm1{k1}{z}, ymm2, ymm3/m256/m32bcst | B     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate ymm1 and ymm2, extract result shifted to the left by value in xmm3/m256 into ymm1. |
| EVEX.512.66.0F38.W0 71 /r VPSHLDVD zmm1{k1}{z}, zmm2, zmm3/m512/m32bcst | B     | V/V                    | AVX512_VBMI2          | Concatenate zmm1 and zmm2, extract result shifted to the left by value in zmm3/m512 into zmm1. |
| EVEX.128.66.0F38.W1 71 /r VPSHLDVQ xmm1{k1}{z}, xmm2, xmm3/m128/m64bcst | B     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate xmm1 and xmm2, extract result shifted to the left by value in xmm3/m128 into xmm1. |
| EVEX.256.66.0F38.W1 71 /r VPSHLDVQ ymm1{k1}{z}, ymm2, ymm3/m256/m64bcst | B     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate ymm1 and ymm2, extract result shifted to the left by value in xmm3/m256 into ymm1. |
| EVEX.512.66.0F38.W1 71 /r VPSHLDVQ zmm1{k1}{z}, zmm2, zmm3/m512/m64bcst | B     | V/V                    | AVX512_VBMI2          | Concatenate zmm1 and zmm2, extract result shifted to the left by value in zmm3/m512 into zmm1. |

## Instruction Operand Encoding

| Op/En | Tuple    | Operand 1        | Operand 2     | Operand 3     | Operand 4 |
| ----- | -------- | ---------------- | ------------- | ------------- | --------- |
| A     | Full Mem | ModRM:reg (r, w) | EVEX.vvvv (r) | ModRM:r/m (r) | N/A       |
| B     | Full     | ModRM:reg (r, w) | EVEX.vvvv (r) | ModRM:r/m (r) | N/A       |

### Description

Concatenate packed data, extract result shifted to the left by variable value.

This instruction supports memory fault suppression.

### Operation

```
FUNCTION concat(a,b):
    IF words:
        d.word[1] := a
        d.word[0] := b
        return d
    ELSE IF dwords:
        q.dword[1] := a
        q.dword[0] := b
        return q
    ELSE IF qwords:
        o.qword[1] := a
        o.qword[0] := b
        return o

```

#### VPSHLDVW DEST, SRC2, SRC3

```
(KL, VL) = (8, 128), (16, 256), (32, 512)
FOR j := 0 TO KL-1:
    IF MaskBit(j) OR *no writemask*:
        tmp := concat(DEST.word[j], SRC2.word[j]) << (SRC3.word[j] & 15)
        DEST.word[j] := tmp.word[1]
    ELSE IF *zeroing*:
        DEST.word[j] := 0
    *ELSE DEST.word[j] remains unchanged*
DEST[MAX_VL-1:VL] := 0

```

#### VPSHLDVD DEST, SRC2, SRC3

```
(KL, VL) = (4, 128), (8, 256), (16, 512)
FOR j := 0 TO KL-1:
    IF SRC3 is broadcast memop:
        tsrc3 := SRC3.dword[0]
    ELSE:
        tsrc3 := SRC3.dword[j]
    IF MaskBit(j) OR *no writemask*:
        tmp := concat(DEST.dword[j], SRC2.dword[j]) << (tsrc3 & 31)
        DEST.dword[j] := tmp.dword[1]
    ELSE IF *zeroing*:
        DEST.dword[j] := 0
    *ELSE DEST.dword[j] remains unchanged*
DEST[MAX_VL-1:VL] := 0

```

#### VPSHLDVQ DEST, SRC2, SRC3

```
(KL, VL) = (2, 128), (4, 256), (8, 512)
FOR j := 0 TO KL-1:
    IF SRC3 is broadcast memop:
        tsrc3 := SRC3.qword[0]
    ELSE:
        tsrc3 := SRC3.qword[j]
    IF MaskBit(j) OR *no writemask*:
        tmp := concat(DEST.qword[j], SRC2.qword[j]) << (tsrc3 & 63)
        DEST.qword[j] := tmp.qword[1]
    ELSE IF *zeroing*:
        DEST.qword[j] := 0
    *ELSE DEST.qword[j] remains unchanged*
DEST[MAX_VL-1:VL] := 0

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
VPSHLDVW __m128i _mm_shldv_epi16(__m128i, __m128i, __m128i);

```

```
VPSHLDVW __m128i _mm_mask_shldv_epi16(__m128i, __mmask8, __m128i, __m128i);

```

```
VPSHLDVW __m128i _mm_maskz_shldv_epi16(__mmask8, __m128i, __m128i, __m128i);

```

```
VPSHLDVW __m256i _mm256_shldv_epi16(__m256i, __m256i, __m256i);

```

```
VPSHLDVW __m256i _mm256_mask_shldv_epi16(__m256i, __mmask16, __m256i, __m256i);

```

```
VPSHLDVW __m256i _mm256_maskz_shldv_epi16(__mmask16, __m256i, __m256i, __m256i);

```

```
VPSHLDVQ __m512i _mm512_shldv_epi64(__m512i, __m512i, __m512i);

```

```
VPSHLDVQ __m512i _mm512_mask_shldv_epi64(__m512i, __mmask8, __m512i, __m512i);

```

```
VPSHLDVQ __m512i _mm512_maskz_shldv_epi64(__mmask8, __m512i, __m512i, __m512i);

```

```
VPSHLDVW __m128i _mm_shldv_epi16(__m128i, __m128i, __m128i);

```

```
VPSHLDVW __m128i _mm_mask_shldv_epi16(__m128i, __mmask8, __m128i, __m128i);

```

```
VPSHLDVW __m128i _mm_maskz_shldv_epi16(__mmask8, __m128i, __m128i, __m128i);

```

```
VPSHLDVW __m256i _mm256_shldv_epi16(__m256i, __m256i, __m256i);

```

```
VPSHLDVW __m256i _mm256_mask_shldv_epi16(__m256i, __mmask16, __m256i, __m256i);

```

```
VPSHLDVW __m256i _mm256_maskz_shldv_epi16(__mmask16, __m256i, __m256i, __m256i);

```

```
VPSHLDVW __m512i _mm512_shldv_epi16(__m512i, __m512i, __m512i);

```

```
VPSHLDVW __m512i _mm512_mask_shldv_epi16(__m512i, __mmask32, __m512i, __m512i);

```

```
VPSHLDVW __m512i _mm512_maskz_shldv_epi16(__mmask32, __m512i, __m512i, __m512i);

```

```
VPSHLDVD __m128i _mm_shldv_epi32(__m128i, __m128i, __m128i);

```

```
VPSHLDVD __m128i _mm_mask_shldv_epi32(__m128i, __mmask8, __m128i, __m128i);

```

```
VPSHLDVD __m128i _mm_maskz_shldv_epi32(__mmask8, __m128i, __m128i, __m128i);

```

```
VPSHLDVD __m256i _mm256_shldv_epi32(__m256i, __m256i, __m256i);

```

```
VPSHLDVD __m256i _mm256_mask_shldv_epi32(__m256i, __mmask8, __m256i, __m256i);

```

```
VPSHLDVD __m256i _mm256_maskz_shldv_epi32(__mmask8, __m256i, __m256i, __m256i);

```

```
VPSHLDVD __m512i _mm512_shldv_epi32(__m512i, __m512i, __m512i);

```

```
VPSHLDVD __m512i _mm512_mask_shldv_epi32(__m512i, __mmask16, __m512i, __m512i);

```

```
VPSHLDVD __m512i _mm512_maskz_shldv_epi32(__mmask16, __m512i, __m512i, __m512i);

```

### SIMD Floating-Point Exceptions

None.

### Other Exceptions

See Table 2-49, “Type E4 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
