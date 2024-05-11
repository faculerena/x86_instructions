# VPEXPANDB/VPEXPANDW**Expand Byte**

| Opcode/Instruction                                    | Op/En | 64/32 bit Mode Support | CPUID Feature Flag    | Description                                                                               |
| ----------------------------------------------------- | ----- | ---------------------- | --------------------- | ----------------------------------------------------------------------------------------- |
| EVEX.128.66.0F38.W0 62 /r VPEXPANDB xmm1{k1}{z}, m128 | A     | V/V                    | AVX512_VBMI2 AVX512VL | Expands up to 128 bits of packed byte values from m128 to xmm1 with writemask k1.         |
| EVEX.128.66.0F38.W0 62 /r VPEXPANDB xmm1{k1}{z}, xmm2 | B     | V/V                    | AVX512_VBMI2 AVX512VL | Expands up to 128 bits of packed byte values from xmm2 to xmm1 with writemask k1.         |
| EVEX.256.66.0F38.W0 62 /r VPEXPANDB ymm1{k1}{z}, m256 | A     | V/V                    | AVX512_VBMI2 AVX512VL | Expands up to 256 bits of packed byte values from m256 to ymm1 with writemask k1.         |
| EVEX.256.66.0F38.W0 62 /r VPEXPANDB ymm1{k1}{z}, ymm2 | B     | V/V                    | AVX512_VBMI2 AVX512VL | Expands up to 256 bits of packed byte values from ymm2 to ymm1 with writemask k1.         |
| EVEX.512.66.0F38.W0 62 /r VPEXPANDB zmm1{k1}{z}, m512 | A     | V/V                    | AVX512_VBMI2          | Expands up to 512 bits of packed byte values from m512 to zmm1 with writemask k1.         |
| EVEX.512.66.0F38.W0 62 /r VPEXPANDB zmm1{k1}{z}, zmm2 | B     | V/V                    | AVX512_VBMI2          | Expands up to 512 bits of packed byte values from zmm2 to zmm1 with writemask k1.         |
| EVEX.128.66.0F38.W1 62 /r VPEXPANDW xmm1{k1}{z}, m128 | A     | V/V                    | AVX512_VBMI2 AVX512VL | Expands up to 128 bits of packed word values from m128 to xmm1 with writemask k1.         |
| EVEX.128.66.0F38.W1 62 /r VPEXPANDW xmm1{k1}{z}, xmm2 | B     | V/V                    | AVX512_VBMI2 AVX512VL | Expands up to 128 bits of packed word values from xmm2 to xmm1 with writemask k1.         |
| EVEX.256.66.0F38.W1 62 /r VPEXPANDW ymm1{k1}{z}, m256 | A     | V/V                    | AVX512_VBMI2 AVX512VL | Expands up to 256 bits of packed word values from m256 to ymm1 with writemask k1.         |
| EVEX.256.66.0F38.W1 62 /r VPEXPANDW ymm1{k1}{z}, ymm2 | B     | V/V                    | AVX512_VBMI2 AVX512VL | Expands up to 256 bits of packed word values from ymm2 to ymm1 with writemask k1.         |
| EVEX.512.66.0F38.W1 62 /r VPEXPANDW zmm1{k1}{z}, m512 | A     | V/V                    | AVX512_VBMI2          | Expands up to 512 bits of packed word values from m512 to zmm1 with writemask k1.         |
| EVEX.512.66.0F38.W1 62 /r VPEXPANDW zmm1{k1}{z}, zmm2 | B     | V/V                    | AVX512_VBMI2          | Expands up to 512 bits of packed byte integer values from zmm2 to zmm1 with writemask k1. |

## Instruction Operand Encoding

| Op/En | Tuple         | Operand 1     | Operand 2     | Operand 3 | Operand 4 |
| ----- | ------------- | ------------- | ------------- | --------- | --------- |
| A     | Tuple1 Scalar | ModRM:reg (w) | ModRM:r/m (r) | N/A       | N/A       |
| B     | N/A           | ModRM:reg (w) | ModRM:r/m (r) | N/A       | N/A       |

### Description

Expands (loads) up to 64 byte integer values or 32 word integer values from the source operand (memory operand) to the destination operand (register operand), based on the active elements determined by the writemask operand.

Note: EVEX.vvvv is reserved and must be 1111b otherwise instructions will #​​​UD.

Moves 128, 256 or 512 bits of packed byte integer values from the source operand (memory operand) to the destination operand (register operand). This instruction is used to load from an int8 vector register or memory location while inserting the data into sparse elements of destination vector register using the active elements pointed out by the operand writemask.

This instruction supports memory fault suppression.

Note that the compressed displacement assumes a pre-scaling (N) corresponding to the size of one single element instead of the size of the full vector.

### Operation

#### VPEXPANDB

```
(KL, VL) = (16, 128), (32, 256), (64, 512)
k := 0
FOR j := 0 TO KL-1:
    IF k1[j] OR *no writemask*:
        DEST.byte[j] := SRC.byte[k];
        k := k + 1
        ELSE:
            IF *merging-masking*:
                *DEST.byte[j] remains unchanged*
                ELSE:
                        ; zeroing-masking
                    DEST.byte[j] := 0
DEST[MAX_VL-1:VL] := 0

```

#### VPEXPANDW

```
(KL, VL) = (8,128), (16,256), (32, 512)
k := 0
FOR j := 0 TO KL-1:
    IF k1[j] OR *no writemask*:
        DEST.word[j] := SRC.word[k];
        k := k + 1
        ELSE:
            IF *merging-masking*:
                *DEST.word[j] remains unchanged*
                ELSE: ; zeroing-masking
                    DEST.word[j] := 0
DEST[MAX_VL-1:VL] := 0

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
VPEXPAND __m128i _mm_mask_expand_epi8(__m128i, __mmask16, __m128i);

```

```
VPEXPAND __m128i _mm_maskz_expand_epi8(__mmask16, __m128i);

```

```
VPEXPAND __m128i _mm_mask_expandloadu_epi8(__m128i, __mmask16, const void*);

```

```
VPEXPAND __m128i _mm_maskz_expandloadu_epi8(__mmask16, const void*);

```

```
VPEXPAND __m256i _mm256_mask_expand_epi8(__m256i, __mmask32, __m256i);

```

```
VPEXPAND __m256i _mm256_maskz_expand_epi8(__mmask32, __m256i);

```

```
VPEXPAND __m256i _mm256_mask_expandloadu_epi8(__m256i, __mmask32, const void*);

```

```
VPEXPAND __m256i _mm256_maskz_expandloadu_epi8(__mmask32, const void*);

```

```
VPEXPAND __m512i _mm512_mask_expand_epi8(__m512i, __mmask64, __m512i);

```

```
VPEXPAND __m512i _mm512_maskz_expand_epi8(__mmask64, __m512i);

```

```
VPEXPAND __m512i _mm512_mask_expandloadu_epi8(__m512i, __mmask64, const void*);

```

```
VPEXPAND __m512i _mm512_maskz_expandloadu_epi8(__mmask64, const void*);

```

```
VPEXPANDW __m128i _mm_mask_expand_epi16(__m128i, __mmask8, __m128i);

```

```
VPEXPANDW __m128i _mm_maskz_expand_epi16(__mmask8, __m128i);

```

```
VPEXPANDW __m128i _mm_mask_expandloadu_epi16(__m128i, __mmask8, const void*);

```

```
VPEXPANDW __m128i _mm_maskz_expandloadu_epi16(__mmask8, const void *);

```

```
VPEXPANDW __m256i _mm256_mask_expand_epi16(__m256i, __mmask16, __m256i);

```

```
VPEXPANDW __m256i _mm256_maskz_expand_epi16(__mmask16, __m256i);

```

```
VPEXPANDW __m256i _mm256_mask_expandloadu_epi16(__m256i, __mmask16, const void*);

```

```
VPEXPANDW __m256i _mm256_maskz_expandloadu_epi16(__mmask16, const void*);

```

```
VPEXPANDW __m512i _mm512_mask_expand_epi16(__m512i, __mmask32, __m512i);

```

```
VPEXPANDW __m512i _mm512_maskz_expand_epi16(__mmask32, __m512i);

```

```
VPEXPANDW __m512i _mm512_mask_expandloadu_epi16(__m512i, __mmask32, const void*);

```

```
VPEXPANDW __m512i _mm512_maskz_expandloadu_epi16(__mmask32, const void*);

```

### SIMD Floating-Point Exceptions

None.

### Other Exceptions

See Table 2-49, “Type E4 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
