# XORPS**Bitwise Logical XOR of Packed Single Precision Floating**

| Opcode/Instruction                                                | Op / En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                                                                            |
| ----------------------------------------------------------------- | ------- | ---------------------- | ------------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| NP 0F 57 /r XORPS xmm1, xmm2/m128                                 | A       | V/V                    | SSE                | Return the bitwise logical XOR of packed single-precision floating-point values in xmm1 and xmm2/mem.                                  |
| VEX.128.0F.WIG 57 /r VXORPS xmm1,xmm2, xmm3/m128                  | B       | V/V                    | AVX                | Return the bitwise logical XOR of packed single-precision floating-point values in xmm2 and xmm3/mem.                                  |
| VEX.256.0F.WIG 57 /r VXORPS ymm1, ymm2, ymm3/m256                 | B       | V/V                    | AVX                | Return the bitwise logical XOR of packed single-precision floating-point values in ymm2 and ymm3/mem.                                  |
| EVEX.128.0F.W0 57 /r VXORPS xmm1 {k1}{z}, xmm2, xmm3/m128/m32bcst | C       | V/V                    | AVX512VL AVX512DQ  | Return the bitwise logical XOR of packed single-precision floating-point values in xmm2 and xmm3/m128/m32bcst subject to writemask k1. |
| EVEX.256.0F.W0 57 /r VXORPS ymm1 {k1}{z}, ymm2, ymm3/m256/m32bcst | C       | V/V                    | AVX512VL AVX512DQ  | Return the bitwise logical XOR of packed single-precision floating-point values in ymm2 and ymm3/m256/m32bcst subject to writemask k1. |
| EVEX.512.0F.W0 57 /r VXORPS zmm1 {k1}{z}, zmm2, zmm3/m512/m32bcst | C       | V/V                    | AVX512DQ           | Return the bitwise logical XOR of packed single-precision floating-point values in zmm2 and zmm3/m512/m32bcst subject to writemask k1. |

## Instruction Operand Encoding

| Op/En | Tuple Type | Operand 1        | Operand 2     | Operand 3     | Operand 4 |
| ----- | ---------- | ---------------- | ------------- | ------------- | --------- |
| A     | N/A        | ModRM:reg (r, w) | ModRM:r/m (r) | N/A           | N/A       |
| B     | N/A        | ModRM:reg (w)    | VEX.vvvv (r)  | ModRM:r/m (r) | N/A       |
| C     | Full       | ModRM:reg (w)    | EVEX.vvvv (r) | ModRM:r/m (r) | N/A       |

### Description

Performs a bitwise logical XOR of the four, eight or sixteen packed single-precision floating-point values from the first source operand and the second source operand, and stores the result in the destination operand

EVEX.512 encoded version: The first source operand is a ZMM register. The second source operand can be a ZMM register or a vector memory location. The destination operand is a ZMM register conditionally updated with write-mask k1.

VEX.256 and EVEX.256 encoded versions: The first source operand is a YMM register. The second source operand is a YMM register or a 256-bit memory location. The destination operand is a YMM register (conditionally updated with writemask k1 in case of EVEX). The upper bits (MAXVL-1:256) of the corresponding ZMM register destination are zeroed.

VEX.128 and EVEX.128 encoded versions: The first source operand is an XMM register. The second source operand is an XMM register or 128-bit memory location. The destination operand is an XMM register (conditionally updated with writemask k1 in case of EVEX). The upper bits (MAXVL-1:128) of the corresponding ZMM register destination are zeroed.

128-bit Legacy SSE version: The second source can be an XMM register or an 128-bit memory location. The destination is not distinct from the first source XMM register and the upper bits (MAXVL-1:128) of the corresponding register destination are unmodified.

### Operation

#### VXORPS (EVEX Encoded Versions)

```
(KL, VL) = (4, 128), (8, 256), (16, 512)
FOR j := 0 TO KL-1
    i := j * 32
    IF k1[j] OR *no writemask* THEN
            IF (EVEX.b == 1) AND (SRC2 *is memory*)
                THEN DEST[i+31:i] := SRC1[i+31:i] BITWISE XOR SRC2[31:0];
                ELSE DEST[i+31:i] := SRC1[i+31:i] BITWISE XOR SRC2[i+31:i];
            FI;
        ELSE
            IF *merging-masking* ; merging-masking
                THEN *DEST[i+31:i] remains unchanged*
                ELSE *zeroing-masking*
                        ; zeroing-masking
                    DEST[i+31:i] = 0
            FI
    FI;
ENDFOR
DEST[MAXVL-1:VL] := 0

```

#### VXORPS (VEX.256 Encoded Version)

```
DEST[31:0] := SRC1[31:0] BITWISE XOR SRC2[31:0]
DEST[63:32] := SRC1[63:32] BITWISE XOR SRC2[63:32]
DEST[95:64] := SRC1[95:64] BITWISE XOR SRC2[95:64]
DEST[127:96] := SRC1[127:96] BITWISE XOR SRC2[127:96]
DEST[159:128] := SRC1[159:128] BITWISE XOR SRC2[159:128]
DEST[191:160] := SRC1[191:160] BITWISE XOR SRC2[191:160]
DEST[223:192] := SRC1[223:192] BITWISE XOR SRC2[223:192]
DEST[255:224] := SRC1[255:224] BITWISE XOR SRC2[255:224].
DEST[MAXVL-1:256] := 0

```

#### VXORPS (VEX.128 Encoded Version)

```
DEST[31:0] := SRC1[31:0] BITWISE XOR SRC2[31:0]
DEST[63:32] := SRC1[63:32] BITWISE XOR SRC2[63:32]
DEST[95:64] := SRC1[95:64] BITWISE XOR SRC2[95:64]
DEST[127:96] := SRC1[127:96] BITWISE XOR SRC2[127:96]
DEST[MAXVL-1:128] := 0

```

#### XORPS (128-bit Legacy SSE Version)

```
DEST[31:0] := SRC1[31:0] BITWISE XOR SRC2[31:0]
DEST[63:32] := SRC1[63:32] BITWISE XOR SRC2[63:32]
DEST[95:64] := SRC1[95:64] BITWISE XOR SRC2[95:64]
DEST[127:96] := SRC1[127:96] BITWISE XOR SRC2[127:96]
DEST[MAXVL-1:128] (Unmodified)

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
VXORPS __m512 _mm512_xor_ps (__m512 a, __m512 b);

```

```
VXORPS __m512 _mm512_mask_xor_ps (__m512 a, __mmask16 m, __m512 b);

```

```
VXORPS __m512 _mm512_maskz_xor_ps (__mmask16 m, __m512 a);

```

```
VXORPS __m256 _mm256_xor_ps (__m256 a, __m256 b);

```

```
VXORPS __m256 _mm256_mask_xor_ps (__m256 a, __mmask8 m, __m256 b);

```

```
VXORPS __m256 _mm256_maskz_xor_ps (__mmask8 m, __m256 a);

```

```
XORPS __m128 _mm_xor_ps (__m128 a, __m128 b);

```

```
VXORPS __m128 _mm_mask_xor_ps (__m128 a, __mmask8 m, __m128 b);

```

```
VXORPS __m128 _mm_maskz_xor_ps (__mmask8 m, __m128 a);

```

### SIMD Floating-Point Exceptions

None.

### Other Exceptions

Non-EVEX-encoded instructions, see Table 2-21, “Type 4 Class Exception Conditions.”

EVEX-encoded instructions, see Table 2-49, “Type E4 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
