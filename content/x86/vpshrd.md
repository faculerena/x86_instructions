#VPSHRD
**Concatenate and Shift Packed Data Right Logical**

| Opcode/Instruction                                                               | Op/En | 64/32 bit Mode Support | CPUID Feature Flag    | Description                                                                                                           |
| -------------------------------------------------------------------------------- | ----- | ---------------------- | --------------------- | --------------------------------------------------------------------------------------------------------------------- |
| EVEX.128.66.0F3A.W1 72 /r /ib VPSHRDW xmm1{k1}{z}, xmm2, xmm3/m128, imm8         | A     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate destination and source operands, extract result shifted to the right by constant value in imm8 into xmm1. |
| EVEX.256.66.0F3A.W1 72 /r /ib VPSHRDW ymm1{k1}{z}, ymm2, ymm3/m256, imm8         | A     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate destination and source operands, extract result shifted to the right by constant value in imm8 into ymm1. |
| EVEX.512.66.0F3A.W1 72 /r /ib VPSHRDW zmm1{k1}{z}, zmm2, zmm3/m512, imm8         | A     | V/V                    | AVX512_VBMI2          | Concatenate destination and source operands, extract result shifted to the right by constant value in imm8 into zmm1. |
| EVEX.128.66.0F3A.W0 73 /r /ib VPSHRDD xmm1{k1}{z}, xmm2, xmm3/m128/m32bcst, imm8 | B     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate destination and source operands, extract result shifted to the right by constant value in imm8 into xmm1. |
| EVEX.256.66.0F3A.W0 73 /r /ib VPSHRDD ymm1{k1}{z}, ymm2, ymm3/m256/m32bcst, imm8 | B     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate destination and source operands, extract result shifted to the right by constant value in imm8 into ymm1. |
| EVEX.512.66.0F3A.W0 73 /r /ib VPSHRDD zmm1{k1}{z}, zmm2, zmm3/m512/m32bcst, imm8 | B     | V/V                    | AVX512_VBMI2          | Concatenate destination and source operands, extract result shifted to the right by constant value in imm8 into zmm1. |
| EVEX.128.66.0F3A.W1 73 /r /ib VPSHRDQ xmm1{k1}{z}, xmm2, xmm3/m128/m64bcst, imm8 | B     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate destination and source operands, extract result shifted to the right by constant value in imm8 into xmm1. |
| EVEX.256.66.0F3A.W1 73 /r /ib VPSHRDQ ymm1{k1}{z}, ymm2, ymm3/m256/m64bcst, imm8 | B     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate destination and source operands, extract result shifted to the right by constant value in imm8 into ymm1. |
| EVEX.512.66.0F3A.W1 73 /r /ib VPSHRDQ zmm1{k1}{z}, zmm2, zmm3/m512/m64bcst, imm8 | B     | V/V                    | AVX512_VBMI2          | Concatenate destination and source operands, extract result shifted to the right by constant value in imm8 into zmm1. |

## Instruction Operand Encoding

| Op/En | Tuple    | Operand 1     | Operand 2     | Operand 3     | Operand 4 |
| ----- | -------- | ------------- | ------------- | ------------- | --------- |
| A     | Full Mem | ModRM:reg (w) | EVEX.vvvv (r) | ModRM:r/m (r) | imm8 (r)  |
| B     | Full     | ModRM:reg (w) | EVEX.vvvv (r) | ModRM:r/m (r) | imm8 (r)  |

### Description

Concatenate packed data, extract result shifted to the right by constant value.

This instruction supports memory fault suppression.

### Operation

#### VPSHRDW DEST, SRC2, SRC3, imm8

```
(KL, VL) = (8, 128), (16, 256), (32, 512)
FOR j := 0 TO KL-1:
    IF MaskBit(j) OR *no writemask*:
        DEST.word[j] := concat(SRC3.word[j], SRC2.word[j]) >> (imm8 & 15)
    ELSE IF *zeroing*:
        DEST.word[j] := 0
    *ELSE DEST.word[j] remains unchanged*
DEST[MAX_VL-1:VL] := 0

```

#### VPSHRDD DEST, SRC2, SRC3, imm8

```
(KL, VL) = (4, 128), (8, 256), (16, 512)
FOR j := 0 TO KL-1:
    IF SRC3 is broadcast memop:
        tsrc3 := SRC3.dword[0]
    ELSE:
        tsrc3 := SRC3.dword[j]
    IF MaskBit(j) OR *no writemask*:
        DEST.dword[j] := concat(tsrc3, SRC2.dword[j]) >> (imm8 & 31)
    ELSE IF *zeroing*:
        DEST.dword[j] := 0
    *ELSE DEST.dword[j] remains unchanged*
DEST[MAX_VL-1:VL] := 0

```

#### VPSHRDQ DEST, SRC2, SRC3, imm8

```
(KL, VL) = (2, 128), (4, 256), (8, 512)
FOR j := 0 TO KL-1:
    IF SRC3 is broadcast memop:
        tsrc3 := SRC3.qword[0]
    ELSE:
        tsrc3 := SRC3.qword[j]
    IF MaskBit(j) OR *no writemask*:
        DEST.qword[j] := concat(tsrc3, SRC2.qword[j]) >> (imm8 & 63)
    ELSE IF *zeroing*:
        DEST.qword[j] := 0
    *ELSE DEST.qword[j] remains unchanged*
DEST[MAX_VL-1:VL] := 0

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
VPSHRDQ __m128i _mm_shrdi_epi64(__m128i, __m128i, int);

```

```
VPSHRDQ __m128i _mm_mask_shrdi_epi64(__m128i, __mmask8, __m128i, __m128i, int);

```

```
VPSHRDQ __m128i _mm_maskz_shrdi_epi64(__mmask8, __m128i, __m128i, int);

```

```
VPSHRDQ __m256i _mm256_shrdi_epi64(__m256i, __m256i, int);

```

```
VPSHRDQ __m256i _mm256_mask_shrdi_epi64(__m256i, __mmask8, __m256i, __m256i, int);

```

```
VPSHRDQ __m256i _mm256_maskz_shrdi_epi64(__mmask8, __m256i, __m256i, int);

```

```
VPSHRDQ __m512i _mm512_shrdi_epi64(__m512i, __m512i, int);

```

```
VPSHRDQ __m512i _mm512_mask_shrdi_epi64(__m512i, __mmask8, __m512i, __m512i, int);

```

```
VPSHRDQ __m512i _mm512_maskz_shrdi_epi64(__mmask8, __m512i, __m512i, int);

```

```
VPSHRDD __m128i _mm_shrdi_epi32(__m128i, __m128i, int);

```

```
VPSHRDD __m128i _mm_mask_shrdi_epi32(__m128i, __mmask8, __m128i, __m128i, int);

```

```
VPSHRDD __m128i _mm_maskz_shrdi_epi32(__mmask8, __m128i, __m128i, int);

```

```
VPSHRDD __m256i _mm256_shrdi_epi32(__m256i, __m256i, int);

```

```
VPSHRDD __m256i _mm256_mask_shrdi_epi32(__m256i, __mmask8, __m256i, __m256i, int);

```

```
VPSHRDD __m256i _mm256_maskz_shrdi_epi32(__mmask8, __m256i, __m256i, int);

```

```
VPSHRDD __m512i _mm512_shrdi_epi32(__m512i, __m512i, int);

```

```
VPSHRDD __m512i _mm512_mask_shrdi_epi32(__m512i, __mmask16, __m512i, __m512i, int);

```

```
VPSHRDD __m512i _mm512_maskz_shrdi_epi32(__mmask16, __m512i, __m512i, int);

```

```
VPSHRDW __m128i _mm_shrdi_epi16(__m128i, __m128i, int);

```

```
VPSHRDW __m128i _mm_mask_shrdi_epi16(__m128i, __mmask8, __m128i, __m128i, int);

```

```
VPSHRDW __m128i _mm_maskz_shrdi_epi16(__mmask8, __m128i, __m128i, int);

```

```
VPSHRDW __m256i _mm256_shrdi_epi16(__m256i, __m256i, int);

```

```
VPSHRDW __m256i _mm256_mask_shrdi_epi16(__m256i, __mmask16, __m256i, __m256i, int);

```

```
VPSHRDW __m256i _mm256_maskz_shrdi_epi16(__mmask16, __m256i, __m256i, int);

```

```
VPSHRDW __m512i _mm512_shrdi_epi16(__m512i, __m512i, int);

```

```
VPSHRDW __m512i _mm512_mask_shrdi_epi16(__m512i, __mmask32, __m512i, __m512i, int);

```

```
VPSHRDW __m512i _mm512_maskz_shrdi_epi16(__mmask32, __m512i, __m512i, int);

```

### SIMD Floating-Point Exceptions

None.

### Other Exceptions

See Table 2-49, “Type E4 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
