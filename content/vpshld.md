# VPSHLD

**Concatenate and Shift Packed Data Left Logical**

| Opcode/Instruction                                                               | Op/En | 64/32 bit Mode Support | CPUID Feature Flag    | Description                                                                                                          |
| -------------------------------------------------------------------------------- | ----- | ---------------------- | --------------------- | -------------------------------------------------------------------------------------------------------------------- |
| EVEX.128.66.0F3A.W1 70 /r /ib VPSHLDW xmm1{k1}{z}, xmm2, xmm3/m128, imm8         | A     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate destination and source operands, extract result shifted to the left by constant value in imm8 into xmm1. |
| EVEX.256.66.0F3A.W1 70 /r /ib VPSHLDW ymm1{k1}{z}, ymm2, ymm3/m256, imm8         | A     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate destination and source operands, extract result shifted to the left by constant value in imm8 into ymm1. |
| EVEX.512.66.0F3A.W1 70 /r /ib VPSHLDW zmm1{k1}{z}, zmm2, zmm3/m512, imm8         | A     | V/V                    | AVX512_VBMI2          | Concatenate destination and source operands, extract result shifted to the left by constant value in imm8 into zmm1. |
| EVEX.128.66.0F3A.W0 71 /r /ib VPSHLDD xmm1{k1}{z}, xmm2, xmm3/m128/m32bcst, imm8 | B     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate destination and source operands, extract result shifted to the left by constant value in imm8 into xmm1. |
| EVEX.256.66.0F3A.W0 71 /r /ib VPSHLDD ymm1{k1}{z}, ymm2, ymm3/m256/m32bcst, imm8 | B     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate destination and source operands, extract result shifted to the left by constant value in imm8 into ymm1. |
| EVEX.512.66.0F3A.W0 71 /r /ib VPSHLDD zmm1{k1}{z}, zmm2, zmm3/m512/m32bcst, imm8 | B     | V/V                    | AVX512_VBMI2          | Concatenate destination and source operands, extract result shifted to the left by constant value in imm8 into zmm1. |
| EVEX.128.66.0F3A.W1 71 /r /ib VPSHLDQ xmm1{k1}{z}, xmm2, xmm3/m128/m64bcst, imm8 | B     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate destination and source operands, extract result shifted to the left by constant value in imm8 into xmm1. |
| EVEX.256.66.0F3A.W1 71 /r /ib VPSHLDQ ymm1{k1}{z}, ymm2, ymm3/m256/m64bcst, imm8 | B     | V/V                    | AVX512_VBMI2 AVX512VL | Concatenate destination and source operands, extract result shifted to the left by constant value in imm8 into ymm1. |
| EVEX.512.66.0F3A.W1 71 /r /ib VPSHLDQ zmm1{k1}{z}, zmm2, zmm3/m512/m64bcst, imm8 | B     | V/V                    | AVX512_VBMI2          | Concatenate destination and source operands, extract result shifted to the left by constant value in imm8 into zmm1. |

## Instruction Operand Encoding

| Op/En | Tuple    | Operand 1     | Operand 2     | Operand 3     | Operand 4 |
| ----- | -------- | ------------- | ------------- | ------------- | --------- |
| A     | Full Mem | ModRM:reg (w) | EVEX.vvvv (r) | ModRM:r/m (r) | imm8 (r)  |
| B     | Full     | ModRM:reg (w) | EVEX.vvvv (r) | ModRM:r/m (r) | imm8 (r)  |

### Description

Concatenate packed data, extract result shifted to the left by constant value.

This instruction supports memory fault suppression.

### Operation

#### VPSHLDW DEST, SRC2, SRC3, imm8

```
(KL, VL) = (8, 128), (16, 256), (32, 512)
FOR j := 0 TO KL-1:
    IF MaskBit(j) OR *no writemask*:
        tmp := concat(SRC2.word[j], SRC3.word[j]) << (imm8 & 15)
        DEST.word[j] := tmp.word[1]
    ELSE IF *zeroing*:
        DEST.word[j] := 0
    *ELSE DEST.word[j] remains unchanged*
DEST[MAX_VL-1:VL] := 0

```

#### VPSHLDD DEST, SRC2, SRC3, imm8

```
(KL, VL) = (4, 128), (8, 256), (16, 512)
FOR j := 0 TO KL-1:
    IF SRC3 is broadcast memop:
        tsrc3 := SRC3.dword[0]
    ELSE:
        tsrc3 := SRC3.dword[j]
    IF MaskBit(j) OR *no writemask*:
        tmp := concat(SRC2.dword[j], tsrc3) << (imm8 & 31)
        DEST.dword[j] := tmp.dword[1]
    ELSE IF *zeroing*:
        DEST.dword[j] := 0
    *ELSE DEST.dword[j] remains unchanged*
DEST[MAX_VL-1:VL] := 0

```

#### VPSHLDQ DEST, SRC2, SRC3, imm8

```
(KL, VL) = (2, 128), (4, 256), (8, 512)
FOR j := 0 TO KL-1:
    IF SRC3 is broadcast memop:
        tsrc3 := SRC3.qword[0]
    ELSE:
        tsrc3 := SRC3.qword[j]
    IF MaskBit(j) OR *no writemask*:
        tmp := concat(SRC2.qword[j], tsrc3) << (imm8 & 63)
        DEST.qword[j] := tmp.qword[1]
    ELSE IF *zeroing*:
        DEST.qword[j] := 0
    *ELSE DEST.qword[j] remains unchanged*
DEST[MAX_VL-1:VL] := 0

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
VPSHLDD __m128i _mm_shldi_epi32(__m128i, __m128i, int);

```

```
VPSHLDD __m128i _mm_mask_shldi_epi32(__m128i, __mmask8, __m128i, __m128i, int);

```

```
VPSHLDD __m128i _mm_maskz_shldi_epi32(__mmask8, __m128i, __m128i, int);

```

```
VPSHLDD __m256i _mm256_shldi_epi32(__m256i, __m256i, int);

```

```
VPSHLDD __m256i _mm256_mask_shldi_epi32(__m256i, __mmask8, __m256i, __m256i, int);

```

```
VPSHLDD __m256i _mm256_maskz_shldi_epi32(__mmask8, __m256i, __m256i, int);

```

```
VPSHLDD __m512i _mm512_shldi_epi32(__m512i, __m512i, int);

```

```
VPSHLDD __m512i _mm512_mask_shldi_epi32(__m512i, __mmask16, __m512i, __m512i, int);

```

```
VPSHLDD __m512i _mm512_maskz_shldi_epi32(__mmask16, __m512i, __m512i, int);

```

```
VPSHLDQ __m128i _mm_shldi_epi64(__m128i, __m128i, int);

```

```
VPSHLDQ __m128i _mm_mask_shldi_epi64(__m128i, __mmask8, __m128i, __m128i, int);

```

```
VPSHLDQ __m128i _mm_maskz_shldi_epi64(__mmask8, __m128i, __m128i, int);

```

```
VPSHLDQ __m256i _mm256_shldi_epi64(__m256i, __m256i, int);

```

```
VPSHLDQ __m256i _mm256_mask_shldi_epi64(__m256i, __mmask8, __m256i, __m256i, int);

```

```
VPSHLDQ __m256i _mm256_maskz_shldi_epi64(__mmask8, __m256i, __m256i, int);

```

```
VPSHLDQ __m512i _mm512_shldi_epi64(__m512i, __m512i, int);

```

```
VPSHLDQ __m512i _mm512_mask_shldi_epi64(__m512i, __mmask8, __m512i, __m512i, int);

```

```
VPSHLDQ __m512i _mm512_maskz_shldi_epi64(__mmask8, __m512i, __m512i, int);

```

```
VPSHLDW __m128i _mm_shldi_epi16(__m128i, __m128i, int);

```

```
VPSHLDW __m128i _mm_mask_shldi_epi16(__m128i, __mmask8, __m128i, __m128i, int);

```

```
VPSHLDW __m128i _mm_maskz_shldi_epi16(__mmask8, __m128i, __m128i, int);

```

```
VPSHLDW __m256i _mm256_shldi_epi16(__m256i, __m256i, int);

```

```
VPSHLDW __m256i _mm256_mask_shldi_epi16(__m256i, __mmask16, __m256i, __m256i, int);

```

```
VPSHLDW __m256i _mm256_maskz_shldi_epi16(__mmask16, __m256i, __m256i, int);

```

```
VPSHLDW __m512i _mm512_shldi_epi16(__m512i, __m512i, int);

```

```
VPSHLDW __m512i _mm512_mask_shldi_epi16(__m512i, __mmask32, __m512i, __m512i, int);

```

```
VPSHLDW __m512i _mm512_maskz_shldi_epi16(__mmask32, __m512i, __m512i, int);

```

### SIMD Floating-Point Exceptions

None.

### Other Exceptions

See Table 2-49, “Type E4 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
