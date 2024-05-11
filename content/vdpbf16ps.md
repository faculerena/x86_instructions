#VDPBF16PS
**Dot Product of BF**

| Opcode/Instruction                                                       | Op/En | 64/32 Bit Mode Support | CPUID Feature Flag   | Description                                                                                                                          |
| ------------------------------------------------------------------------ | ----- | ---------------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| EVEX.128.F3.0F38.W0 52 /r VDPBF16PS xmm1{k1}{z}, xmm2, xmm3/m128/m32bcst | A     | V/V                    | AVX512VL AVX512_BF16 | Multiply BF16 pairs from xmm2 and xmm3/m128, and accumulate the resulting packed single precision results in xmm1 with writemask k1. |
| EVEX.256.F3.0F38.W0 52 /r VDPBF16PS ymm1{k1}{z}, ymm2, ymm3/m256/m32bcst | A     | V/V                    | AVX512VL AVX512_BF16 | Multiply BF16 pairs from ymm2 and ymm3/m256, and accumulate the resulting packed single precision results in ymm1 with writemask k1. |
| EVEX.512.F3.0F38.W0 52 /r VDPBF16PS zmm1{k1}{z}, zmm2, zmm3/m512/m32bcst | A     | V/V                    | AVX512F AVX512_BF16  | Multiply BF16 pairs from zmm2 and zmm3/m512, and accumulate the resulting packed single precision results in zmm1 with writemask k1. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1     | Operand 2     | Operand 3     | Operand 4 |
| ----- | ----- | ------------- | ------------- | ------------- | --------- |
| A     | Full  | ModRM:reg (w) | EVEX.vvvv (r) | ModRM:r/m (r) | N/A       |

### Description

This instruction performs a SIMD dot-product of two BF16 pairs and accumulates into a packed single precision register.

“Round to nearest even” rounding mode is used when doing each accumulation of the FMA. Output denormals are always flushed to zero and input denormals are always treated as zero. MXCSR is not consulted nor updated.

NaN propagation priorities are described in [Table 5-1](/x86/vdpbf16ps#tbl-5-1).

| NaN Priority | Description      | Comments                                                                    |
| ------------ | ---------------- | --------------------------------------------------------------------------- |
| 1            | src1 low is NaN  | Lower part has priority over upper part, i.e., it overrides the upper part. |
| 2            | src2 low is NaN  |
| 3            | src1 high is NaN | Upper part may be overridden if lower has NaN.                              |
| 4            | src2 high is NaN |
| 5            | srcdest is NaN   | Dest is propagated if no NaN is encountered by src2.                        |

[Table 5-1](/x86/vdpbf16ps#tbl-5-1). NaN Propagation Priorities

### Operation

```
Define make_fp32(x):
    // The x parameter is bfloat16. Pack it in to upper 16b of a dword. The bit pattern is a legal fp32 value. Return that bit pattern.
    dword := 0
    dword[31:16] := x
    RETURN dword

```

#### VDPBF16PS srcdest, src1, src2

```
VL = (128, 256, 512)
KL = VL/32
origdest := srcdest
FOR i := 0 to KL-1:
    IF k1[ i ] or *no writemask*:
        IF src2 is memory and evex.b == 1:
            t := src2.dword[0]
        ELSE:
            t := src2.dword[ i ]
        // FP32 FMA with daz in, ftz out and RNE rounding. MXCSR neither consulted nor updated.
        srcdest.fp32[ i ] += make_fp32(src1.bfloat16[2*i+1]) * make_fp32(t.bfloat[1])
        srcdest.fp32[ i ] += make_fp32(src1.bfloat16[2*i+0]) * make_fp32(t.bfloat[0])
    ELSE IF *zeroing*:
        srcdest.dword[ i ] := 0
    ELSE: // merge masking, dest element unchanged
        srcdest.dword[ i ] := origdest.dword[ i ]
srcdest[MAXVL-1:VL] := 0

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
VDPBF16PS __m128 _mm_dpbf16_ps(__m128, __m128bh, __m128bh);

```

```
VDPBF16PS __m128 _mm_mask_dpbf16_ps( __m128, __mmask8, __m128bh, __m128bh);

```

```
VDPBF16PS __m128 _mm_maskz_dpbf16_ps(__mmask8, __m128, __m128bh, __m128bh);

```

```
VDPBF16PS __m256 _mm256_dpbf16_ps(__m256, __m256bh, __m256bh);

```

```
VDPBF16PS __m256 _mm256_mask_dpbf16_ps(__m256, __mmask8, __m256bh, __m256bh);

```

```
VDPBF16PS __m256 _mm256_maskz_dpbf16_ps(__mmask8, __m256, __m256bh, __m256bh);

```

```
VDPBF16PS __m512 _mm512_dpbf16_ps(__m512, __m512bh, __m512bh);

```

```
VDPBF16PS __m512 _mm512_mask_dpbf16_ps(__m512, __mmask16, __m512bh, __m512bh);

```

```
VDPBF16PS __m512 _mm512_maskz_dpbf16_ps(__mmask16, __m512, __m512bh, __m512bh);

```

### SIMD Floating-Point Exceptions

None.

### Other Exceptions

See Table 2-49, “Type E4 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
