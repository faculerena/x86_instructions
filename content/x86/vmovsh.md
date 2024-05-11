# VMOVSH**Move Scalar FP**

| Instruction En bit Mode Flag Support Instruction En bit Mode Flag Support 64/32 CPUID Feature Instruction En bit Mode Flag CPUID Feature Instruction En bit Mode Flag Op/ 64/32 CPUID Feature Instruction En bit Mode Flag 64/32 CPUID Feature Instruction En bit Mode Flag CPUID Feature Instruction En bit Mode Flag Op/ 64/32 CPUID Feature |     | Support |             | Description                                                                                                     |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- | ------- | ----------- | --------------------------------------------------------------------------------------------------------------- |
| EVEX.LLIG.F3.MAP5.W0 10 /r VMOVSH xmm1{k1}{z}, m16                                                                                                                                                                                                                                                                                             | A   | V/V     | AVX512-FP16 | Move FP16 value from m16 to xmm1 subject to writemask k1.                                                       |
| EVEX.LLIG.F3.MAP5.W0 11 /r VMOVSH m16{k1}, xmm1                                                                                                                                                                                                                                                                                                | B   | V/V     | AVX512-FP16 | Move low FP16 value from xmm1 to m16 subject to writemask k1.                                                   |
| EVEX.LLIG.F3.MAP5.W0 10 /r VMOVSH xmm1{k1}{z}, xmm2, xmm3                                                                                                                                                                                                                                                                                      | C   | V/V     | AVX512-FP16 | Move low FP16 values from xmm3 to xmm1 subject to writemask k1. Bits 127:16 of xmm2 are copied to xmm1[127:16]. |
| EVEX.LLIG.F3.MAP5.W0 11 /r VMOVSH xmm1{k1}{z}, xmm2, xmm3                                                                                                                                                                                                                                                                                      | D   | V/V     | AVX512-FP16 | Move low FP16 values from xmm3 to xmm1 subject to writemask k1. Bits 127:16 of xmm2 are copied to xmm1[127:16]. |

## Instruction Operand Encoding

| Op/En | Tuple  | Operand 1     | Operand 2     | Operand 3     | Operand 4 |
| ----- | ------ | ------------- | ------------- | ------------- | --------- |
| A     | Scalar | ModRM:reg (w) | ModRM:r/m (r) | N/A           | N/A       |
| B     | Scalar | ModRM:r/m (w) | ModRM:reg (r) | N/A           | N/A       |
| C     | N/A    | ModRM:reg (w) | VEX.vvvv (r)  | ModRM:r/m (r) | N/A       |
| D     | N/A    | ModRM:r/m (w) | VEX.vvvv (r)  | ModRM:reg (r) | N/A       |

### Description

This instruction moves a FP16 value to a register or memory location.

The two register-only forms are aliases and differ only in where their operands are encoded; this is a side effect of the encodings selected.

### Operation

#### VMOVSH dest, src (two operand load)

```
IF k1[0] or no writemask:
    DEST.fp16[0] := SRC.fp16[0]
ELSE IF *zeroing*:
    DEST.fp16[0] := 0
// ELSE DEST.fp16[0] remains unchanged
DEST[MAXVL:16] := 0

```

#### VMOVSH dest, src (two operand store)

```
IF k1[0] or no writemask:
    DEST.fp16[0] := SRC.fp16[0]
// ELSE DEST.fp16[0] remains unchanged

```

#### VMOVSH dest, src1, src2 (three operand copy)

```
IF k1[0] or no writemask:
    DEST.fp16[0] := SRC2.fp16[0]
ELSE IF *zeroing*:
    DEST.fp16[0] := 0
// ELSE DEST.fp16[0] remains unchanged
DEST[127:16] := SRC1[127:16]
DEST[MAXVL:128] := 0

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
VMOVSH __m128h _mm_load_sh (void const* mem_addr);

```

```
VMOVSH __m128h _mm_mask_load_sh (__m128h src, __mmask8 k, void const* mem_addr);

```

```
VMOVSH __m128h _mm_maskz_load_sh (__mmask8 k, void const* mem_addr);

```

```
VMOVSH __m128h _mm_mask_move_sh (__m128h src, __mmask8 k, __m128h a, __m128h b);

```

```
VMOVSH __m128h _mm_maskz_move_sh (__mmask8 k, __m128h a, __m128h b);

```

```
VMOVSH __m128h _mm_move_sh (__m128h a, __m128h b);

```

```
VMOVSH void _mm_mask_store_sh (void * mem_addr, __mmask8 k, __m128h a);

```

```
VMOVSH void _mm_store_sh (void * mem_addr, __m128h a);

```

### SIMD Floating-Point Exceptions

None

### Other Exceptions

EVEX-encoded instruction, see Table 2-51, “Type E5 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
