# VGETEXPPH**Convert Exponents of Packed FP**

| Instruction En Bit Mode Flag Support Instruction En Bit Mode Flag Support 64/32 CPUID Feature Instruction En Bit Mode Flag CPUID Feature Instruction En Bit Mode Flag Op/ 64/32 CPUID Feature Instruction En Bit Mode Flag 64/32 CPUID Feature Instruction En Bit Mode Flag CPUID Feature Instruction En Bit Mode Flag Op/ 64/32 CPUID Feature |     | Support |                      | Description                                                                                                                                                                                   |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- | ------- | -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| EVEX.128.66.MAP6.W0 42 /r VGETEXPPH xmm1{k1}{z}, xmm2/m128/m16bcst                                                                                                                                                                                                                                                                             | A   | V/V     | AVX512-FP16 AVX512VL | Convert the exponent of FP16 values in the source operand to FP16 results representing unbiased integer exponents and stores the results in the destination register subject to writemask k1. |
| EVEX.256.66.MAP6.W0 42 /r VGETEXPPH ymm1{k1}{z}, ymm2/m256/m16bcst                                                                                                                                                                                                                                                                             | A   | V/V     | AVX512-FP16 AVX512VL | Convert the exponent of FP16 values in the source operand to FP16 results representing unbiased integer exponents and stores the results in the destination register subject to writemask k1. |
| EVEX.512.66.MAP6.W0 42 /r VGETEXPPH zmm1{k1}{z}, zmm2/m512/m16bcst {sae}                                                                                                                                                                                                                                                                       | A   | V/V     | AVX512-FP16          | Convert the exponent of FP16 values in the source operand to FP16 results representing unbiased integer exponents and stores the results in the destination register subject to writemask k1. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1     | Operand 2     | Operand 3 | Operand 4 |
| ----- | ----- | ------------- | ------------- | --------- | --------- |
| A     | Full  | ModRM:reg (w) | ModRM:r/m (r) | N/A       | N/A       |

### Description

This instruction extracts the biased exponents from the normalized FP16 representation of each word element of the source operand (the second operand) as unbiased signed integer value, or convert the denormal representation of input data to unbiased negative integer values. Each integer value of the unbiased exponent is converted to an FP16 value and written to the corresponding word elements of the destination operand (the first operand) as FP16 numbers.

The destination elements are updated according to the writemask.

Each GETEXP operation converts the exponent value into a floating-point number (permitting input value in denormal representation). Special cases of input values are listed in [Table 5-6](/x86/vfmsub132ph:vfnmsub132ph:vfmsub213ph:vfnmsub213ph:vfmsub231ph:vfnmsub231ph#tbl-5-6).

The formula is:

GETEXP(x) = floor(log2(|x|))

Notation **floor(x)** stands for maximal integer not exceeding real number x.

Software usage of VGETEXPxx and VGETMANTxx instructions generally involve a combination of GETEXP operation and GETMANT operation (see VGETMANTPH). Thus, the VGETEXPPH instruction does not require software to handle SIMD floating-point exceptions.

| Input Operand | Result     | Comments                                                       |
| ------------- | ---------- | -------------------------------------------------------------- | ----------- | ---- | --- |
| src1 = NaN    | QNaN(src1) | If (SRC = SNaN), then #​​IE. If (SRC = denormal), then #​​​DE. |
| 0 <           | src1       | < INF                                                          | floor(log2( | src1 | ))  |
|               | src1       | = +INF                                                         | +INF        |
|               | src1       | = 0                                                            | -INF        |

[Table 5-16](/x86/vgetexpph#tbl-5-16). VGETEXPPH/VGETEXPSH Special Cases

### Operation

```
def normalize_exponent_tiny_fp16(src):
    jbit := 0
    // src & dst are FP16 numbers with sign(1b), exp(5b) and fraction (10b) fields
    dst.exp := 1 // write bits 14:10
    dst.fraction := src.fraction // copy bits 9:0
    while jbit == 0:
        jbit := dst.fraction[9] // msb of the fraction
        dst.fraction := dst.fraction << 1
        dst.exp := dst.exp - 1
    dst.fraction := 0
    return dst
def getexp_fp16(src):
    src.sign := 0 // make positive
    exponent_all_ones := (src[14:10] == 0x1F)
    exponent_all_zeros := (src[14:10] == 0)
    mantissa_all_zeros := (src[9:0] == 0)
    zero := exponent_all_zeros and mantissa_all_zeros
    signaling_bit := src[9]
    nan := exponent_all_ones and not(mantissa_all_zeros)
    snan := nan and not(signaling_bit)
    qnan := nan and signaling_bit
    positive_infinity := not(negative) and exponent_all_ones and mantissa_all_zeros
    denormal := exponent_all_zeros and not(mantissa_all_zeros)
    if nan:
        if snan:
            MXCSR.IE := 1
        return qnan(src)
                // convert snan to a qnan
    if positive_infinity:
        return src
    if zero:
        return -INF
    if denormal:
        tmp := normalize_exponent_tiny_fp16(src)
        MXCSR.DE := 1
    else:
        tmp := src
    tmp := SAR(tmp, 10) // shift arithmetic right
    tmp := tmp - 15 // subtract bias
    return convert_integer_to_fp16(tmp)

```

#### VGETEXPPH dest{k1}, src

```
VL = 128, 256 or 512
KL := VL/16
FOR i := 0 to KL-1:
    IF k1[i] or *no writemask*:
        IF SRC is memory and (EVEX.b = 1):
            tsrc := src.fp16[0]
        ELSE:
            tsrc := src.fp16[i]
        DEST.fp16[i] := getexp_fp16(tsrc)
    ELSE IF *zeroing*:
        DEST.fp16[i] := 0
    //else DEST.fp16[i] remains unchanged
DEST[MAXVL-1:VL] := 0

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
VGETEXPPH __m128h _mm_getexp_ph (__m128h a);

```

```
VGETEXPPH __m128h _mm_mask_getexp_ph (__m128h src, __mmask8 k, __m128h a);

```

```
VGETEXPPH __m128h _mm_maskz_getexp_ph (__mmask8 k, __m128h a);

```

```
VGETEXPPH __m256h _mm256_getexp_ph (__m256h a);

```

```
VGETEXPPH __m256h _mm256_mask_getexp_ph (__m256h src, __mmask16 k, __m256h a);

```

```
VGETEXPPH __m256h _mm256_maskz_getexp_ph (__mmask16 k, __m256h a);

```

```
VGETEXPPH __m512h _mm512_getexp_ph (__m512h a);

```

```
VGETEXPPH __m512h _mm512_mask_getexp_ph (__m512h src, __mmask32 k, __m512h a);

```

```
VGETEXPPH __m512h _mm512_maskz_getexp_ph (__mmask32 k, __m512h a);

```

```
VGETEXPPH __m512h _mm512_getexp_round_ph (__m512h a, const int sae);

```

```
VGETEXPPH __m512h _mm512_mask_getexp_round_ph (__m512h src, __mmask32 k, __m512h a, const int sae);

```

```
VGETEXPPH __m512h _mm512_maskz_getexp_round_ph (__mmask32 k, __m512h a, const int sae);

```

### SIMD Floating-Point Exceptions

Invalid, Denormal.

### Other Exceptions

EVEX-encoded instructions, see Table 2-46, “Type E2 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
