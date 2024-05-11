#VREDUCEPH
**Perform Reduction Transformation on Packed FP**

| Instruction En bit Mode Flag Support Instruction En bit Mode Flag Support 64/32 CPUID Feature Instruction En bit Mode Flag CPUID Feature Instruction En bit Mode Flag Op/ 64/32 CPUID Feature Instruction En bit Mode Flag 64/32 CPUID Feature Instruction En bit Mode Flag CPUID Feature Instruction En bit Mode Flag Op/ 64/32 CPUID Feature |     | Support |                      | Description                                                                                                                                                                                         |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- | ------- | -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| EVEX.128.NP.0F3A.W0 56 /r /ib VREDUCEPH xmm1{k1}{z}, xmm2/m128/m16bcst, imm8                                                                                                                                                                                                                                                                   | A   | V/V     | AVX512-FP16 AVX512VL | Perform reduction transformation on packed FP16 values in xmm2/m128/m16bcst by subtracting a number of fraction bits specified by the imm8 field. Store the result in xmm1 subject to writemask k1. |
| EVEX.256.NP.0F3A.W0 56 /r /ib VREDUCEPH ymm1{k1}{z}, ymm2/m256/m16bcst, imm8                                                                                                                                                                                                                                                                   | A   | V/V     | AVX512-FP16 AVX512VL | Perform reduction transformation on packed FP16 values in ymm2/m256/m16bcst by subtracting a number of fraction bits specified by the imm8 field. Store the result in ymm1 subject to writemask k1. |
| EVEX.512.NP.0F3A.W0 56 /r /ib VREDUCEPH zmm1{k1}{z}, zmm2/m512/m16bcst {sae}, imm8                                                                                                                                                                                                                                                             | A   | V/V     | AVX512-FP16          | Perform reduction transformation on packed FP16 values in zmm2/m512/m16bcst by subtracting a number of fraction bits specified by the imm8 field. Store the result in zmm1 subject to writemask k1. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1     | Operand 2     | Operand 3 | Operand 4 |
| ----- | ----- | ------------- | ------------- | --------- | --------- |
| A     | Full  | ModRM:reg (w) | ModRM:r/m (r) | imm8 (r)  | N/A       |

### Description

This instruction performs a reduction transformation of the packed binary encoded FP16 values in the source operand (the second operand) and store the reduced results in binary FP format to the destination operand (the first operand) under the writemask k1.

The reduction transformation subtracts the integer part and the leading M fractional bits from the binary FP source value, where M is a unsigned integer specified by imm8[7:4]. Specifically, the reduction transformation can be expressed as:

dest = src − (ROUND(2M \* src)) \* 2−M

where ROUND() treats src, 2M, and their product as binary FP numbers with normalized significand and biased exponents.

The magnitude of the reduced result can be expressed by considering src = 2p \* man2, where ‘man2’ is the normalized significand and ‘p’ is the unbiased exponent.

Then if RC=RNE: 0 ≤ |ReducedResult| ≤ 2−M−1.

Then if RC ≠ RNE: 0 ≤ |ReducedResult| < 2−M.

This instruction might end up with a precision exception set. However, in case of SPE set (i.e., Suppress Precision Exception, which is imm8[3]=1), no precision exception is reported.

This instruction may generate tiny non-zero result. If it does so, it does not report underflow exception, even if underflow exceptions are unmasked (UM flag in MXCSR register is 0).

For special cases, see [Table 5-30](/x86/vreduceph#tbl-5-30).

| Input value                       | Round Mode          | Returned Value |
| --------------------------------- | ------------------- | -------------- | ------------ | -------------------- |
|                                   | Src1                | < 2*−*M*−*1    | RNE          | Src1                 |
|                                   | Src1                | < 2*−M*        | RU, Src1 > 0 | Round(Src1 − 2*−*M)1 |
| RU, Src1 ≤ 0                      | Src1                |
| RD, Src1 ≥ 0                      | Src1                |
| RD, Src1 < 0                      | Round(Src1 + 2*−*M) |
| Src1 = ±0 or Dest = ±0 (Src1 ≠ ∞) | NOT RD              | +0.0           |
| RD                                | −0.0                |
| Src1 = ±∞                         | Any                 | +0.0           |
| Src1 = ±NAN                       | Any                 | QNaN (Src1)    |

[Table 5-30](/x86/vreduceph#tbl-5-30). VREDUCEPH/VREDUCESH Special Cases

> 1. The Round(.) function uses rounding controls specified by (imm8[2]? MXCSR.RC: imm8[1:0]).

### Operation

```
def reduce_fp16(src, imm8):
    nan := (src.exp = 0x1F) and (src.fraction != 0)
    if nan:
        return QNAN(src)
    m := imm8[7:4]
    rc := imm8[1:0]
    rc_source := imm8[2]
    spe := imm[3] // suppress precision exception
    tmp := 2^(-m) * ROUND(2^m * src, spe, rc_source, rc)
    tmp := src - tmp // using same RC, SPE controls
    return tmp

```

#### VREDUCEPH dest{k1}, src, imm8

```
VL = 128, 256 or 512
KL := VL/16
FOR i := 0 to KL-1:
    IF k1[i] or *no writemask*:
        IF SRC is memory and (EVEX.b = 1):
            tsrc := src.fp16[0]
        ELSE:
            tsrc := src.fp16[i]
        DEST.fp16[i] := reduce_fp16(tsrc, imm8)
    ELSE IF *zeroing*:
        DEST.fp16[i] := 0
    //else DEST.fp16[i] remains unchanged
DEST[MAXVL-1:VL] := 0

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
VREDUCEPH __m128h _mm_mask_reduce_ph (__m128h src, __mmask8 k, __m128h a, int imm8);

```

```
VREDUCEPH __m128h _mm_maskz_reduce_ph (__mmask8 k, __m128h a, int imm8);

```

```
VREDUCEPH __m128h _mm_reduce_ph (__m128h a, int imm8);

```

```
VREDUCEPH __m256h _mm256_mask_reduce_ph (__m256h src, __mmask16 k, __m256h a, int imm8);

```

```
VREDUCEPH __m256h _mm256_maskz_reduce_ph (__mmask16 k, __m256h a, int imm8);

```

```
VREDUCEPH __m256h _mm256_reduce_ph (__m256h a, int imm8);

```

```
VREDUCEPH __m512h _mm512_mask_reduce_ph (__m512h src, __mmask32 k, __m512h a, int imm8);

```

```
VREDUCEPH __m512h _mm512_maskz_reduce_ph (__mmask32 k, __m512h a, int imm8);

```

```
VREDUCEPH __m512h _mm512_reduce_ph (__m512h a, int imm8);

```

```
VREDUCEPH __m512h _mm512_mask_reduce_round_ph (__m512h src, __mmask32 k, __m512h a, int imm8, const int sae);

```

```
VREDUCEPH __m512h _mm512_maskz_reduce_round_ph (__mmask32 k, __m512h a, int imm8, const int sae);

```

```
VREDUCEPH __m512h _mm512_reduce_round_ph (__m512h a, int imm8, const int sae);

```

### SIMD Floating-Point Exceptions

Invalid, Precision.

### Other Exceptions

EVEX-encoded instruction, see Table 2-46, “Type E2 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
