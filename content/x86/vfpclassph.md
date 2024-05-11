#VFPCLASSPH
**Test Types of Packed FP**

| Instruction En Bit Mode Flag Support Instruction En Bit Mode Flag Support 64/32 CPUID Feature Instruction En Bit Mode Flag CPUID Feature Instruction En Bit Mode Flag Op/ 64/32 CPUID Feature Instruction En Bit Mode Flag 64/32 CPUID Feature Instruction En Bit Mode Flag CPUID Feature Instruction En Bit Mode Flag Op/ 64/32 CPUID Feature |     | Support |                      | Description                                                                                                                                                                                                                                     |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- | ------- | -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| EVEX.128.NP.0F3A.W0 66 /r /ib VFPCLASSPH k1{k2}, xmm1/m128/m16bcst, imm8                                                                                                                                                                                                                                                                       | A   | V/V     | AVX512-FP16 AVX512VL | Test the input for the following categories: NaN, +0, -0, +Infinity, -Infinity, denormal, finite negative. The immediate field provides a mask bitforeachofthesecategorytests. Themasked test results are OR-ed together to form a mask result. |
| EVEX.256.NP.0F3A.W0 66 /r /ib VFPCLASSPH k1{k2}, ymm1/m256/m16bcst, imm8                                                                                                                                                                                                                                                                       | A   | V/V     | AVX512-FP16 AVX512VL | Test the input for the following categories: NaN, +0, -0, +Infinity, -Infinity, denormal, finite negative. The immediate field provides a mask bitforeachofthesecategorytests. Themasked test results are OR-ed together to form a mask result. |
| EVEX.512.NP.0F3A.W0 66 /r /ib VFPCLASSPH k1{k2}, zmm1/m512/m16bcst, imm8                                                                                                                                                                                                                                                                       | A   | V/V     | AVX512-FP16          | Test the input for the following categories: NaN, +0, -0, +Infinity, -Infinity, denormal, finite negative. The immediate field provides a mask bitforeachofthesecategorytests. Themasked test results are OR-ed together to form a mask result. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1     | Operand 2     | Operand 3 | Operand 4 |
| ----- | ----- | ------------- | ------------- | --------- | --------- |
| A     | Full  | ModRM:reg (w) | ModRM:r/m (r) | imm8 (r)  | N/A       |

### Description

This instruction checks the packed FP16 values in the source operand for special categories, specified by the set bits in the imm8 byte. Each set bit in imm8 specifies a category of floating-point values that the input data element is classified against; see [Table 5-9](/x86/vfpclassph#tbl-5-9) for the categories. The classified results of all specified categories of an input value are ORed together to form the final boolean result for the input element. The result is written to the corresponding bits in the destination mask register according to the writemask.

| Bits    | Category | Classifier                 |
| ------- | -------- | -------------------------- |
| imm8[0] | QNAN     | Checks for QNAN            |
| imm8[1] | PosZero  | Checks +0                  |
| imm8[2] | NegZero  | Checks for -0              |
| imm8[3] | PosINF   | Checks for +∞              |
| imm8[4] | NegINF   | Checks for −∞              |
| imm8[5] | Denormal | Checks for Denormal        |
| imm8[6] | Negative | Checks for Negative finite |
| imm8[7] | SNAN     | Checks for SNAN            |

[Table 5-9](/x86/vfpclassph#tbl-5-9). Classifier Operations for VFPCLASSPH/VFPCLASSSH

### Operation

```
def check_fp_class_fp16(tsrc, imm8):
    negative := tsrc[15]
    exponent_all_ones := (tsrc[14:10] == 0x1F)
    exponent_all_zeros := (tsrc[14:10] == 0)
    mantissa_all_zeros := (tsrc[9:0] == 0)
    zero := exponent_all_zeros and mantissa_all_zeros
    signaling_bit := tsrc[9]
    snan := exponent_all_ones and not(mantissa_all_zeros) and not(signaling_bit)
    qnan := exponent_all_ones and not(mantissa_all_zeros) and signaling_bit
    positive_zero := not(negative) and zero
    negative_zero := negative and zero
    positive_infinity := not(negative) and exponent_all_ones and mantissa_all_zeros
    negative_infinity := negative and exponent_all_ones and mantissa_all_zeros
    denormal := exponent_all_zeros and not(mantissa_all_zeros)
    finite_negative := negative and not(exponent_all_ones) and not(zero)
    return (imm8[0] and qnan) OR
        (imm8[1] and positive_zero) OR
        (imm8[2] and negative_zero) OR
        (imm8[3] and positive_infinity) OR
        (imm8[4] and negative_infinity) OR
        (imm8[5] and denormal) OR
        (imm8[6] and finite_negative) OR
        (imm8[7] and snan)

```

#### VFPCLASSPH dest{k2}, src, imm8

```
VL = 128, 256 or 512
KL := VL/16
FOR i := 0 to KL-1:
    IF k2[i] or *no writemask*:
        IF SRC is memory and (EVEX.b = 1):
            tsrc := SRC.fp16[0]
        ELSE:
            tsrc := SRC.fp16[i]
        DEST.bit[i] := check_fp_class_fp16(tsrc, imm8)
    ELSE:
        DEST.bit[i] := 0
DEST[MAXKL-1:kl] := 0

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
VFPCLASSPH __mmask8 _mm_fpclass_ph_mask (__m128h a, int imm8);

```

```
VFPCLASSPH __mmask8 _mm_mask_fpclass_ph_mask (__mmask8 k1, __m128h a, int imm8);

```

```
VFPCLASSPH __mmask16 _mm256_fpclass_ph_mask (__m256h a, int imm8);

```

```
VFPCLASSPH __mmask16 _mm256_mask_fpclass_ph_mask (__mmask16 k1, __m256h a, int imm8);

```

```
VFPCLASSPH __mmask32 _mm512_fpclass_ph_mask (__m512h a, int imm8);

```

```
VFPCLASSPH __mmask32 _mm512_mask_fpclass_ph_mask (__mmask32 k1, __m512h a, int imm8);

```

### SIMD Floating-Point Exceptions

None.

### Other Exceptions

EVEX-encoded instructions, see Table 2-49, “Type E4 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
