# VCVTPS2PHX

**Convert Packed Single Precision Floating**

| Opcode/Instruction                                                       | Op / En | 64/32 Bit Mode Support | CPUID Feature Flag   | Description                                                                                                                                                      |
| ------------------------------------------------------------------------ | ------- | ---------------------- | -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| EVEX.128.66.MAP5.W0 1D /r VCVTPS2PHX xmm1{k1}{z}, xmm2/m128/m32bcst      | A       | V/V                    | AVX512-FP16 AVX512VL | Convert four packed single precision floating-point values in xmm2/m128/m32bcst to packed FP16 values, and store the result in xmm1 subject to writemask k1.     |
| EVEX.256.66.MAP5.W0 1D /r VCVTPS2PHX xmm1{k1}{z}, ymm2/m256/m32bcst      | A       | V/V                    | AVX512-FP16 AVX512VL | Convert eight packed single precision floating-point values in ymm2/m256/m32bcst to packed FP16 values, and store the result in xmm1 subject to writemask k1.    |
| EVEX.512.66.MAP5.W0 1D /r VCVTPS2PHX ymm1{k1}{z}, zmm2/m512/m32bcst {er} | A       | V/V                    | AVX512-FP16          | Convert sixteen packed single precision floating-point values in zmm2 /m512/m32bcst to packed FP16 values, and store the result in ymm1 subject to writemask k1. |

## Instruction Operand Encoding

| Op/En | Tuple Type | Operand 1     | Operand 2     | Operand 3 | Operand 4 |
| ----- | ---------- | ------------- | ------------- | --------- | --------- |
| A     | Full       | ModRM:reg (w) | ModRM:r/m (r) | N/A       | N/A       |

### Description

This instruction converts packed single precision floating values in the source operand to FP16 values and stores to the destination operand.

The VCVTPS2PHX instruction supports broadcasting.

This instruction uses MXCSR.DAZ for handling FP32 inputs. FP16 outputs can be normal or denormal numbers, and are not conditionally flushed based on MXCSR settings.

### Operation

#### VCVTPS2PHX DEST, SRC (AVX512_FP16 Load Version With Broadcast Support)

```
VL = 128, 256, or 512
KL := VL / 32
IF *SRC is a register* and (VL == 512) and (EVEX.b = 1):
    SET_RM(EVEX.RC)
ELSE:
    SET_RM(MXCSR.RC)
FOR j := 0 TO KL-1:
    IF k1[j] OR *no writemask*:
        IF *SRC is memory* and EVEX.b = 1:
            tsrc := SRC.fp32[0]
        ELSE
            tsrc := SRC.fp32[j]
        DEST.fp16[j] := Convert_fp32_to_fp16(tsrc)
    ELSE IF *zeroing*:
        DEST.fp16[j] := 0
    // else dest.fp16[j] remains unchanged
DEST[MAXVL-1:VL/2] := 0

```

### Flags Affected

None.

### Intel C/C++ Compiler Intrinsic Equivalent

```
VCVTPS2PHX __m256h _mm512_cvtx_roundps_ph (__m512 a, int rounding);

```

```
VCVTPS2PHX __m256h _mm512_mask_cvtx_roundps_ph (__m256h src, __mmask16 k, __m512 a, int rounding);

```

```
VCVTPS2PHX __m256h _mm512_maskz_cvtx_roundps_ph (__mmask16 k, __m512 a, int rounding);

```

```
VCVTPS2PHX __m128h _mm_cvtxps_ph (__m128 a);

```

```
VCVTPS2PHX __m128h _mm_mask_cvtxps_ph (__m128h src, __mmask8 k, __m128 a);

```

```
VCVTPS2PHX __m128h _mm_maskz_cvtxps_ph (__mmask8 k, __m128 a);

```

```
VCVTPS2PHX __m128h _mm256_cvtxps_ph (__m256 a);

```

```
VCVTPS2PHX __m128h _mm256_mask_cvtxps_ph (__m128h src, __mmask8 k, __m256 a);

```

```
VCVTPS2PHX __m128h _mm256_maskz_cvtxps_ph (__mmask8 k, __m256 a);

```

```
VCVTPS2PHX __m256h _mm512_cvtxps_ph (__m512 a);

```

```
VCVTPS2PHX __m256h _mm512_mask_cvtxps_ph (__m256h src, __mmask16 k, __m512 a);

```

```
VCVTPS2PHX __m256h _mm512_maskz_cvtxps_ph (__mmask16 k, __m512 a);

```

### SIMD Floating-Point Exceptions

Invalid, Underflow, Overflow, Precision, Denormal (if MXCSR.DAZ=0).

### Other Exceptions

EVEX-encoded instructions, see Table 2-46, “Type E2 Class Exception Conditions.”

Additionally:

| #​​​UD | If VEX.W=1.                                 |
| ------ | ------------------------------------------- |
| #​​​UD | If VEX.vvvv != 1111B or EVEX.vvvv != 1111B. |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
