# VRNDSCALEPH**Round Packed FP**

| Instruction En bit Mode Flag Support Instruction En bit Mode Flag Support 64/32 CPUID Feature Instruction En bit Mode Flag CPUID Feature Instruction En bit Mode Flag Op/ 64/32 CPUID Feature Instruction En bit Mode Flag 64/32 CPUID Feature Instruction En bit Mode Flag CPUID Feature Instruction En bit Mode Flag Op/ 64/32 CPUID Feature |     | Support |                      | Description                                                                                                                                               |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- | ------- | -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| EVEX.128.NP.0F3A.W0 08 /r /ib VRNDSCALEPH xmm1{k1}{z}, xmm2/m128/m16bcst, imm8                                                                                                                                                                                                                                                                 | A   | V/V     | AVX512-FP16 AVX512VL | Round packed FP16 values in xmm2/m128/m16bcst to a number of fraction bits specified by the imm8 field. Store the result in xmm1 subject to writemask k1. |
| EVEX.256.NP.0F3A.W0 08 /r /ib VRNDSCALEPH ymm1{k1}{z}, ymm2/m256/m16bcst, imm8                                                                                                                                                                                                                                                                 | A   | V/V     | AVX512-FP16 AVX512VL | Round packed FP16 values in ymm2/m256/m16bcst to a number of fraction bits specified by the imm8 field. Store the result in ymm1 subject to writemask k1. |
| EVEX.512.NP.0F3A.W0 08 /r /ib VRNDSCALEPH zmm1{k1}{z}, zmm2/m512/m16bcst {sae}, imm8                                                                                                                                                                                                                                                           | A   | V/V     | AVX512-FP16          | Round packed FP16 values in zmm2/m512/m16bcst to a number of fraction bits specified by the imm8 field. Store the result in zmm1 subject to writemask k1. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1     | Operand 2     | Operand 3 | Operand 4 |
| ----- | ----- | ------------- | ------------- | --------- | --------- |
| A     | Full  | ModRM:reg (w) | ModRM:r/m (r) | imm8 (r)  | N/A       |

### Description

This instruction rounds the FP16 values in the source operand by the rounding mode specified in the immediate operand (see [Table 5-32](/x86/vrndscaleph#tbl-5-32)) and places the result in the destination operand. The destination operand is conditionally updated according to the writemask.

The rounding process rounds the input to an integral value, plus number bits of fraction that are specified by imm8[7:4] (to be included in the result), and returns the result as an FP16 value.

Note that no overflow is induced while executing this instruction (although the source is scaled by the imm8[7:4] value).

The immediate operand also specifies control fields for the rounding operation. Three bit fields are defined and shown in [Table 5-32](/x86/vrndscaleph#tbl-5-32), “Imm8 Controls for VRNDSCALEPH/VRNDSCALESH.” Bit 3 of the immediate byte controls the processor behavior for a precision exception, bit 2 selects the source of rounding mode control, and bits 1:0 specify a non-sticky rounding-mode value.

The Precision Floating-Point Exception is signaled according to the immediate operand. If any source operand is an SNaN then it will be converted to a QNaN.

The sign of the result of this instruction is preserved, including the sign of zero. Special cases are described in Table 5-33.

The formula of the operation on each data element for VRNDSCALEPH is

ROUND(x) = 2−M \*Round_to_INT(x \* 2M, round_ctrl),

round_ctrl = imm[3:0];

M=imm[7:4];

The operation of x \* 2M is computed as if the exponent range is unlimited (i.e., no overflow ever occurs).

If this instruction encoding’s SPE bit (bit 3) in the immediate operand is 1, VRNDSCALEPH can set MXCSR.UE without MXCSR.PE.

EVEX.vvvv is reserved and must be 1111b, otherwise instructions will #​​​UD.

| Imm8 Bits | Description                                                                                           |
| --------- | ----------------------------------------------------------------------------------------------------- |
| imm8[7:4] | Number of fixed points to preserve.                                                                   |
| imm8[3]   | Suppress Precision Exception (SPE) 0b00: Implies use of MXCSR exception mask. 0b01: Implies suppress. |
| imm8[2]   | Round Select (RS) 0b00: Implies use of imm8[1:0]. 0b01: Implies use of MXCSR.                         |
| imm8[1:0] | Round Control Override: 0b00: Round nearest even. 0b01: Round down. 0b10: Round up. 0b11: Truncate.   |

[Table 5-32](/x86/vrndscaleph#tbl-5-32). Imm8 Controls for VRNDSCALEPH/VRNDSCALESH

| Input Value | Returned Value         |
| ----------- | ---------------------- |
| Src1 = ±∞   | Src1                   |
| Src1 = ±NaN | Src1 converted to QNaN |
| Src1 = ±0   | Src1                   |

[Table 5-33](/x86/vrndscaleph#tbl-5-33). VRNDSCALEPH/VRNDSCALESH Special Cases

### Operation

```
def round_fp16_to_integer(src, imm8):
    if imm8[2] = 1:
        rounding_direction := MXCSR.RC
    else:
        rounding_direction := imm8[1:0]
    m := imm8[7:4] // scaling factor
    tsrc1 := 2^m * src
    if rounding_direction = 0b00:
        tmp := round_to_nearest_even_integer(trc1)
    else if rounding_direction = 0b01:
        tmp := round_to_equal_or_smaller_integer(trc1)
    else if rounding_direction = 0b10:
        tmp := round_to_equal_or_larger_integer(trc1)
    else if rounding_direction = 0b11:
        tmp := round_to_smallest_magnitude_integer(trc1)
    dst := 2^(-m) * tmp
    if imm8[3]==0: // check SPE
        if src != dst:
            MXCSR.PE := 1
    return dst

```

#### VRNDSCALEPH dest{k1}, src, imm8

```
VL = 128, 256 or 512
KL := VL/16
FOR i := 0 to KL-1:
    IF k1[i] or *no writemask*:
        IF SRC is memory and (EVEX.b = 1):
            tsrc := src.fp16[0]
        ELSE:
            tsrc := src.fp16[i]
        DEST.fp16[i] := round_fp16_to_integer(tsrc, imm8)
    ELSE IF *zeroing*:
        DEST.fp16[i] := 0
    //else DEST.fp16[i] remains unchanged
DEST[MAXVL-1:VL] := 0

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
VRNDSCALEPH __m128h _mm_mask_roundscale_ph (__m128h src, __mmask8 k, __m128h a, int imm8);

```

```
VRNDSCALEPH __m128h _mm_maskz_roundscale_ph (__mmask8 k, __m128h a, int imm8);

```

```
VRNDSCALEPH __m128h _mm_roundscale_ph (__m128h a, int imm8);

```

```
VRNDSCALEPH __m256h _mm256_mask_roundscale_ph (__m256h src, __mmask16 k, __m256h a, int imm8);

```

```
VRNDSCALEPH __m256h _mm256_maskz_roundscale_ph (__mmask16 k, __m256h a, int imm8);

```

```
VRNDSCALEPH __m256h _mm256_roundscale_ph (__m256h a, int imm8);

```

```
VRNDSCALEPH __m512h _mm512_mask_roundscale_ph (__m512h src, __mmask32 k, __m512h a, int imm8);

```

```
VRNDSCALEPH __m512h _mm512_maskz_roundscale_ph (__mmask32 k, __m512h a, int imm8);

```

```
VRNDSCALEPH __m512h _mm512_roundscale_ph (__m512h a, int imm8);

```

```
VRNDSCALEPH __m512h _mm512_mask_roundscale_round_ph (__m512h src, __mmask32 k, __m512h a, int imm8, const int sae);

```

```
VRNDSCALEPH __m512h _mm512_maskz_roundscale_round_ph (__mmask32 k, __m512h a, int imm8, const int sae);

```

```
VRNDSCALEPH __m512h _mm512_roundscale_round_ph (__m512h a, int imm8, const int sae);

```

### SIMD Floating-Point Exceptions

Invalid, Underflow, Precision.

### Other Exceptions

EVEX-encoded instruction, see Table 2-46, “Type E2 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
