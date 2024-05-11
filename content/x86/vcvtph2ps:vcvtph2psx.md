#VCVTPH2PS/VCVTPH2PSX
**Convert Packed FP**

| Opcode/Instruction                                                        | Op / En | 64/32 Bit Mode Support | CPUID Feature Flag   | Description                                                                                                                                                         |
| ------------------------------------------------------------------------- | ------- | ---------------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| VEX.128.66.0F38.W0 13 /r VCVTPH2PS xmm1, xmm2/m64                         | A       | V/V                    | F16C                 | Convert four packed FP16 values in xmm2/m64 to packed single precision floating-point value in xmm1.                                                                |
| VEX.256.66.0F38.W0 13 /r VCVTPH2PS ymm1, xmm2/m128                        | A       | V/V                    | F16C                 | Convert eight packed FP16 values in xmm2/m128 to packed single precision floating-point value in ymm1.                                                              |
| EVEX.128.66.0F38.W0 13 /r VCVTPH2PS xmm1 {k1}{z}, xmm2/m64                | B       | V/V                    | AVX512VL AVX512F     | Convert four packed FP16 values in xmm2/m64 to packed single precision floating-point values in xmm1 subject to writemask k1.                                       |
| EVEX.256.66.0F38.W0 13 /r VCVTPH2PS ymm1 {k1}{z}, xmm2/m128               | B       | V/V                    | AVX512VL AVX512F     | Convert eight packed FP16 values in xmm2/m128 to packed single precision floating-point values in ymm1 subject to writemask k1.                                     |
| EVEX.512.66.0F38.W0 13 /r VCVTPH2PS zmm1 {k1}{z}, ymm2/m256 {sae}         | B       | V/V                    | AVX512F              | Convert sixteen packed FP16 values in ymm2/m256 to packed single precision floating-point values in zmm1 subject to writemask k1.                                   |
| EVEX.128.66.MAP6.W0 13 /r VCVTPH2PSX xmm1{k1}{z}, xmm2/m64/m16bcst        | C       | V/V                    | AVX512-FP16 AVX512VL | Convert four packed FP16 values in xmm2/m64/m16bcst to four packed single precision floating-point values, and store result in xmm1 subject to writemask k1.        |
| EVEX.256.66.MAP6.W0 13 /r VCVTPH2PSX ymm1{k1}{z}, xmm2/m128/m16bcst       | C       | V/V                    | AVX512-FP16 AVX512VL | Convert eight packed FP16 values in xmm2/m128/m16bcst to eight packed single precision floating-point values, and store result in ymm1 subject to writemask k1.     |
| EVEX.512.66.MAP6.W0 13 /r VCVTPH2PSX zmm1{k1}{z}, ymm2/m256/m16bcst {sae} | C       | V/V                    | AVX512-FP16          | Convert sixteen packed FP16 values in ymm2/m256/m16bcst to sixteen packed single precision floating-point values, and store result in zmm1 subject to writemask k1. |

## Instruction Operand Encoding

| Op/En | Tuple Type | Operand 1     | Operand 2     | Operand 3 | Operand 4 |
| ----- | ---------- | ------------- | ------------- | --------- | --------- |
| A     | N/A        | ModRM:reg (w) | ModRM:r/m (r) | N/A       | N/A       |
| B     | Half Mem   | ModRM:reg (w) | ModRM:r/m (r) | N/A       | N/A       |
| C     | Half       | ModRM:reg (w) | ModRM:r/m (r) | N/A       | N/A       |

### Description

This instruction converts packed half precision (16-bits) floating-point values in the low-order bits of the source operand (the second operand) to packed single precision floating-point values and writes the converted values into the destination operand (the first operand).

If case of a denormal operand, the correct normal result is returned. MXCSR.DAZ is ignored and is treated as if it 0. No denormal exception is reported on MXCSR.

VEX.128 version: The source operand is a XMM register or 64-bit memory location. The destination operand is a XMM register. The upper bits (MAXVL-1:128) of the corresponding destination register are zeroed.

VEX.256 version: The source operand is a XMM register or 128-bit memory location. The destination operand is a YMM register. Bits (MAXVL-1:256) of the corresponding destination register are zeroed.

EVEX encoded versions: The source operand is a YMM/XMM/XMM (low 64-bits) register or a 256/128/64-bit memory location. The destination operand is a ZMM/YMM/XMM register conditionally updated with writemask k1.

The diagram below illustrates how data is converted from four packed half precision (in 64 bits) to four single precision (in 128 bits) floating-point values.

Note: VEX.vvvv and EVEX.vvvv are reserved (must be 1111b).

| VCVTPH2PS*xmm1,xmm2/mem64, imm8* 127 96 95 64 63 48 47 32 31 16 15 0 |
| -------------------------------------------------------------------- |
| xmm2/mem64 VH3 VH2 VH1 VH0                                           |
| convert convert convert convert 127 96 95 64 63 32 31 0              |
| VS3 VS2 VS1 VS0 xmm1                                                 |

[Figure 5-6](/x86/vcvtph2ps:vcvtph2psx#fig-5-6). VCVTPH2PS (128-bit Version)

The VCVTPH2PSX instruction is a new form of the PH to PS conversion instruction, encoded in map 6. The previous version of the instruction, VCVTPH2PS, that is present in AVX512F (encoded in map 2, 0F38) does not support embedded broadcasting. The VCVTPH2PSX instruction has the embedded broadcasting option available.

The instructions associated with AVX512_FP16 always handle FP16 denormal number inputs; denormal inputs are not treated as zero.

### Operation

```
vCvt_h2s(SRC1[15:0])
{
RETURN Cvt_Half_Precision_To_Single_Precision(SRC1[15:0]);
}

```

#### VCVTPH2PS (EVEX Encoded Versions)

```
(KL, VL) = (4, 128), (8, 256), (16, 512)
FOR j := 0 TO KL-1
    i := j * 32
    k := j * 16
    IF k1[j] OR *no writemask*
        THEN DEST[i+31:i] :=
            vCvt_h2s(SRC[k+15:k])
        ELSE
            IF *merging-masking*
                        ; merging-masking
                THEN *DEST[i+31:i] remains unchanged*
                ELSE
                        ; zeroing-masking
                    DEST[i+31:i] := 0
            FI
    FI;
ENDFOR
DEST[MAXVL-1:VL] := 0

```

#### VCVTPH2PS (VEX.256 Encoded Version)

```
DEST[31:0] := vCvt_h2s(SRC1[15:0]);
DEST[63:32] := vCvt_h2s(SRC1[31:16]);
DEST[95:64] := vCvt_h2s(SRC1[47:32]);
DEST[127:96] := vCvt_h2s(SRC1[63:48]);
DEST[159:128] := vCvt_h2s(SRC1[79:64]);
DEST[191:160] := vCvt_h2s(SRC1[95:80]);
DEST[223:192] := vCvt_h2s(SRC1[111:96]);
DEST[255:224] := vCvt_h2s(SRC1[127:112]);
DEST[MAXVL-1:256] := 0

```

#### VCVTPH2PS (VEX.128 Encoded Version)

```
DEST[31:0] := vCvt_h2s(SRC1[15:0]);
DEST[63:32] := vCvt_h2s(SRC1[31:16]);
DEST[95:64] := vCvt_h2s(SRC1[47:32]);
DEST[127:96] := vCvt_h2s(SRC1[63:48]);
DEST[MAXVL-1:128] := 0

```

#### VCVTPH2PSX DEST, SRC

```
VL = 128, 256, or 512
KL := VL/32
FOR j := 0 TO KL-1:
    IF k1[j] OR *no writemask*:
        IF *SRC is memory* and EVEX.b = 1:
            tsrc := SRC.fp16[0]
        ELSE
            tsrc := SRC.fp16[j]
        DEST.fp32[j] := Convert_fp16_to_fp32(tsrc)
    ELSE IF *zeroing*:
        DEST.fp32[j] := 0
    // else dest.fp32[j] remains unchanged
DEST[MAXVL-1:VL] := 0

```

### Flags Affected

None.

### Intel C/C++ Compiler Intrinsic Equivalent

```
VCVTPH2PS __m512 _mm512_cvtph_ps( __m256i a);

```

```
VCVTPH2PS __m512 _mm512_mask_cvtph_ps(__m512 s, __mmask16 k, __m256i a);

```

```
VCVTPH2PS __m512 _mm512_maskz_cvtph_ps(__mmask16 k, __m256i a);

```

```
VCVTPH2PS __m512 _mm512_cvt_roundph_ps( __m256i a, int sae);

```

```
VCVTPH2PS __m512 _mm512_mask_cvt_roundph_ps(__m512 s, __mmask16 k, __m256i a, int sae);

```

```
VCVTPH2PS __m512 _mm512_maskz_cvt_roundph_ps( __mmask16 k, __m256i a, int sae);

```

```
VCVTPH2PS __m256 _mm256_mask_cvtph_ps(__m256 s, __mmask8 k, __m128i a);

```

```
VCVTPH2PS __m256 _mm256_maskz_cvtph_ps(__mmask8 k, __m128i a);

```

```
VCVTPH2PS __m128 _mm_mask_cvtph_ps(__m128 s, __mmask8 k, __m128i a);

```

```
VCVTPH2PS __m128 _mm_maskz_cvtph_ps(__mmask8 k, __m128i a);

```

```
VCVTPH2PS __m128 _mm_cvtph_ps ( __m128i m1);

```

```
VCVTPH2PS __m256 _mm256_cvtph_ps ( __m128i m1)

```

```
VCVTPH2PSX __m512 _mm512_cvtx_roundph_ps (__m256h a, int sae);

```

```
VCVTPH2PSX __m512 _mm512_mask_cvtx_roundph_ps (__m512 src, __mmask16 k, __m256h a, int sae);

```

```
VCVTPH2PSX __m512 _mm512_maskz_cvtx_roundph_ps (__mmask16 k, __m256h a, int sae);

```

```
VCVTPH2PSX __m128 _mm_cvtxph_ps (__m128h a);

```

```
VCVTPH2PSX __m128 _mm_mask_cvtxph_ps (__m128 src, __mmask8 k, __m128h a);

```

```
VCVTPH2PSX __m128 _mm_maskz_cvtxph_ps (__mmask8 k, __m128h a);

```

```
VCVTPH2PSX __m256 _mm256_cvtxph_ps (__m128h a);

```

```
VCVTPH2PSX __m256 _mm256_mask_cvtxph_ps (__m256 src, __mmask8 k, __m128h a);

```

```
VCVTPH2PSX __m256 _mm256_maskz_cvtxph_ps (__mmask8 k, __m128h a);

```

```
VCVTPH2PSX __m512 _mm512_cvtxph_ps (__m256h a);

```

```
VCVTPH2PSX __m512 _mm512_mask_cvtxph_ps (__m512 src, __mmask16 k, __m256h a);

```

```
VCVTPH2PSX __m512 _mm512_maskz_cvtxph_ps (__mmask16 k, __m256h a);

```

### SIMD Floating-Point Exceptions

VEX-encoded instructions: Invalid.

EVEX-encoded instructions: Invalid.

EVEX-encoded instructions with broadcast (VCVTPH2PSX): Invalid, Denormal.

### Other Exceptions

VEX-encoded instructions, see Table 2-26, “Type 11 Class Exception Conditions” (do not report #​AC).

EVEX-encoded instructions, see Table 2-60, “Type E11 Class Exception Conditions.”

EVEX-encoded instructions with broadcast (VCVTPH2PSX), see Table 2-46, “Type E2 Class Exception Conditions.”

Additionally:

| #​​​UD | If VEX.W=1.                                 |
| ------ | ------------------------------------------- |
| #​​​UD | If VEX.vvvv != 1111B or EVEX.vvvv != 1111B. |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
