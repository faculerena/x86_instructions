# SUBSD**Subtract Scalar Double Precision Floating**

| Opcode/Instruction                                               | Op / En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                                                                   |
| ---------------------------------------------------------------- | ------- | ---------------------- | ------------------ | ----------------------------------------------------------------------------------------------------------------------------- |
| F2 0F 5C /r SUBSD xmm1, xmm2/m64                                 | A       | V/V                    | SSE2               | Subtract the low double precision floating-point value in xmm2/m64 from xmm1 and store the result in xmm1.                    |
| VEX.LIG.F2.0F.WIG 5C /r VSUBSD xmm1,xmm2, xmm3/m64               | B       | V/V                    | AVX                | Subtract the low double precision floating-point value in xmm3/m64 from xmm2 and store the result in xmm1.                    |
| EVEX.LLIG.F2.0F.W1 5C /r VSUBSD xmm1 {k1}{z}, xmm2, xmm3/m64{er} | C       | V/V                    | AVX512F            | Subtract the low double precision floating-point value in xmm3/m64 from xmm2 and store the result in xmm1 under writemask k1. |

## Instruction Operand Encoding

| Op/En | Tuple Type    | Operand 1        | Operand 2     | Operand 3     | Operand 4 |
| ----- | ------------- | ---------------- | ------------- | ------------- | --------- |
| A     | N/A           | ModRM:reg (r, w) | ModRM:r/m (r) | N/A           | N/A       |
| B     | N/A           | ModRM:reg (w)    | VEX.vvvv (r)  | ModRM:r/m (r) | N/A       |
| C     | Tuple1 Scalar | ModRM:reg (w)    | EVEX.vvvv (r) | ModRM:r/m (r) | N/A       |

## Description

Subtract the low double precision floating-point value in the second source operand from the first source operand and stores the double precision floating-point result in the low quadword of the destination operand.

The second source operand can be an XMM register or a 64-bit memory location. The first source and destination operands are XMM registers.

128-bit Legacy SSE version: The destination and first source operand are the same. Bits (MAXVL-1:64) of the corresponding destination register remain unchanged.

VEX.128 and EVEX encoded versions: Bits (127:64) of the XMM register destination are copied from corresponding bits in the first source operand. Bits (MAXVL-1:128) of the destination register are zeroed.

EVEX encoded version: The low quadword element of the destination operand is updated according to the write-mask.

Software should ensure VSUBSD is encoded with VEX.L=0. Encoding VSUBSD with VEX.L=1 may encounter unpredictable behavior across different processor generations.

## Operation

### VSUBSD (EVEX Encoded Version)

```
IF (SRC2 *is register*) AND (EVEX.b = 1)
    THEN
        SET_ROUNDING_MODE_FOR_THIS_INSTRUCTION(EVEX.RC);
    ELSE
        SET_ROUNDING_MODE_FOR_THIS_INSTRUCTION(MXCSR.RC);
FI;
IF k1[0] or *no writemask*
    THEN DEST[63:0] := SRC1[63:0] - SRC2[63:0]
    ELSE
        IF *merging-masking* ; merging-masking
            THEN *DEST[63:0] remains unchanged*
            ELSE ; zeroing-masking
                THEN DEST[63:0] := 0
        FI;
FI;
DEST[127:64] := SRC1[127:64]
DEST[MAXVL-1:128] := 0

```

### VSUBSD (VEX.128 Encoded Version)

```
DEST[63:0] := SRC1[63:0] - SRC2[63:0]
DEST[127:64] := SRC1[127:64]
DEST[MAXVL-1:128] := 0

```

### SUBSD (128-bit Legacy SSE Version)

```
DEST[63:0] := DEST[63:0] - SRC[63:0]
DEST[MAXVL-1:64] (Unmodified)

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
VSUBSD __m128d _mm_mask_sub_sd (__m128d s, __mmask8 k, __m128d a, __m128d b);

```

```
VSUBSD __m128d _mm_maskz_sub_sd (__mmask8 k, __m128d a, __m128d b);

```

```
VSUBSD __m128d _mm_sub_round_sd (__m128d a, __m128d b, int);

```

```
VSUBSD __m128d _mm_mask_sub_round_sd (__m128d s, __mmask8 k, __m128d a, __m128d b, int);

```

```
VSUBSD __m128d _mm_maskz_sub_round_sd (__mmask8 k, __m128d a, __m128d b, int);

```

```
SUBSD __m128d _mm_sub_sd (__m128d a, __m128d b);

```

## SIMD Floating-Point Exceptions

Overflow, Underflow, Invalid, Precision, Denormal.

## Other Exceptions

VEX-encoded instructions, see Table 2-20, “Type 3 Class Exception Conditions.”

EVEX-encoded instructions, see Table 2-47, “Type E3 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
