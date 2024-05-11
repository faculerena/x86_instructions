# PSHUFB**Packed Shuffle Bytes**

| Opcode/Instruction                                               | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                   |
| ---------------------------------------------------------------- | ----- | ---------------------- | ------------------ | ----------------------------------------------------------------------------- |
| NP 0F 38 00 /r1 PSHUFB mm1, mm2/m64                              | A     | V/V                    | SSSE3              | Shuffle bytes in mm1 according to contents of mm2/m64.                        |
| 66 0F 38 00 /r PSHUFB xmm1, xmm2/m128                            | A     | V/V                    | SSSE3              | Shuffle bytes in xmm1 according to contents of xmm2/m128.                     |
| VEX.128.66.0F38.WIG 00 /r VPSHUFB xmm1, xmm2, xmm3/m128          | B     | V/V                    | AVX                | Shuffle bytes in xmm2 according to contents of xmm3/m128.                     |
| VEX.256.66.0F38.WIG 00 /r VPSHUFB ymm1, ymm2, ymm3/m256          | B     | V/V                    | AVX2               | Shuffle bytes in ymm2 according to contents of ymm3/m256.                     |
| EVEX.128.66.0F38.WIG 00 /r VPSHUFB xmm1 {k1}{z}, xmm2, xmm3/m128 | C     | V/V                    | AVX512VL AVX512BW  | Shuffle bytes in xmm2 according to contents of xmm3/m128 under write mask k1. |
| EVEX.256.66.0F38.WIG 00 /r VPSHUFB ymm1 {k1}{z}, ymm2, ymm3/m256 | C     | V/V                    | AVX512VL AVX512BW  | Shuffle bytes in ymm2 according to contents of ymm3/m256 under write mask k1. |
| EVEX.512.66.0F38.WIG 00 /r VPSHUFB zmm1 {k1}{z}, zmm2, zmm3/m512 | C     | V/V                    | AVX512BW           | Shuffle bytes in zmm2 according to contents of zmm3/m512 under write mask k1. |

> 1. See note in Section 2.5, “Intel® AVX and Intel® SSE Instruction Exception Classification,” in the Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 2A, and Section 23.25.3, “Exception Conditions of Legacy SIMD Instructions Operating on MMX Registers,” in the Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 3B.

## Instruction Operand Encoding

| Op/En | Tuple Type | Operand 1        | Operand 2     | Operand 3     | Operand 4 |
| ----- | ---------- | ---------------- | ------------- | ------------- | --------- |
| A     | N/A        | ModRM:reg (r, w) | ModRM:r/m (r) | N/A           | N/A       |
| B     | N/A        | ModRM:reg (w)    | VEX.vvvv (r)  | ModRM:r/m (r) | N/A       |
| C     | Full Mem   | ModRM:reg (w)    | EVEX.vvvv (r) | ModRM:r/m (r) | N/A       |

## Description

PSHUFB performs in-place shuffles of bytes in the destination operand (the first operand) according to the shuffle control mask in the source operand (the second operand). The instruction permutes the data in the destination operand, leaving the shuffle mask unaffected. If the most significant bit (bit[7]) of each byte of the shuffle control mask is set, then constant zero is written in the result byte. Each byte in the shuffle control mask forms an index to permute the corresponding byte in the destination operand. The value of each index is the least significant 4 bits (128-bit operation) or 3 bits (64-bit operation) of the shuffle control byte. When the source operand is a 128-bit memory operand, the operand must be aligned on a 16-byte boundary or a general-protection exception (#​​​​GP) will be generated.

In 64-bit mode and not encoded with VEX/EVEX, use the REX prefix to access XMM8-XMM15 registers.

Legacy SSE version 64-bit operand: Both operands can be MMX registers.

128-bit Legacy SSE version: The first source operand and the destination operand are the same. Bits (MAXVL-1:128) of the corresponding YMM destination register remain unchanged.

VEX.128 encoded version: The destination operand is the first operand, the first source operand is the second operand, the second source operand is the third operand. Bits (MAXVL-1:128) of the destination YMM register are zeroed.

VEX.256 encoded version: Bits (255:128) of the destination YMM register stores the 16-byte shuffle result of the upper 16 bytes of the first source operand, using the upper 16-bytes of the second source operand as control mask.

The value of each index is for the high 128-bit lane is the least significant 4 bits of the respective shuffle control byte. The index value selects a source data element within each 128-bit lane.

EVEX encoded version: The second source operand is an ZMM/YMM/XMM register or an 512/256/128-bit memory location. The first source operand and destination operands are ZMM/YMM/XMM registers. The destination is conditionally updated with writemask k1.

EVEX and VEX encoded version: Four/two in-lane 128-bit shuffles.

## Operation

### PSHUFB (With 64-bit Operands)

```
TEMP := DEST
for i = 0 to 7 {
    if (SRC[(i * 8)+7] = 1 ) then
            DEST[(i*8)+7...(i*8)+0] := 0;
    else
            index[2..0] := SRC[(i*8)+2 .. (i*8)+0];
            DEST[(i*8)+7...(i*8)+0] := TEMP[(index*8+7)..(index*8+0)];
    endif;
}
PSHUFB (with 128 bit operands)
TEMP := DEST
for i = 0 to 15 {
    if (SRC[(i * 8)+7] = 1 ) then
            DEST[(i*8)+7..(i*8)+0] := 0;
        else
            index[3..0] := SRC[(i*8)+3 .. (i*8)+0];
            DEST[(i*8)+7..(i*8)+0] := TEMP[(index*8+7)..(index*8+0)];
    endif
}

```

### VPSHUFB (VEX.128 Encoded Version)

```
for i = 0 to 15 {
    if (SRC2[(i * 8)+7] = 1) then
        DEST[(i*8)+7..(i*8)+0] := 0;
        else
        index[3..0] := SRC2[(i*8)+3 .. (i*8)+0];
        DEST[(i*8)+7..(i*8)+0] := SRC1[(index*8+7)..(index*8+0)];
    endif
}
DEST[MAXVL-1:128] := 0

```

### VPSHUFB (VEX.256 Encoded Version)

```
for i = 0 to 15 {
    if (SRC2[(i * 8)+7] == 1 ) then
        DEST[(i*8)+7..(i*8)+0] := 0;
        else
        index[3..0] := SRC2[(i*8)+3 .. (i*8)+0];
        DEST[(i*8)+7..(i*8)+0] := SRC1[(index*8+7)..(index*8+0)];
    endif
    if (SRC2[128 + (i * 8)+7] == 1 ) then
        DEST[128 + (i*8)+7..(i*8)+0] := 0;
        else
        index[3..0] := SRC2[128 + (i*8)+3 .. (i*8)+0];
        DEST[128 + (i*8)+7..(i*8)+0] := SRC1[128 + (index*8+7)..(index*8+0)];
    endif
}

```

### VPSHUFB (EVEX Encoded Versions)

```
(KL, VL) = (16, 128), (32, 256), (64, 512)
jmask := (KL-1) & ~0xF
                // 0x00, 0x10, 0x30 depending on the VL
FOR j = 0 TO KL-1
                // dest
    IF kl[ i ] or no_masking
        index := src.byte[ j ];
        IF index & 0x80
            Dest.byte[ j ] := 0;
        ELSE
            index := (index & 0xF) + (j & jmask);
                // 16-element in-lane lookup
            Dest.byte[ j ] := src.byte[ index ];
    ELSE if zeroing
        Dest.byte[ j ] := 0;
DEST[MAXVL-1:VL] := 0;

```

MM2
07H
07H
FFH
80H
01H
00H
00H
00H
MM1
04H
01H
07H
03H
02H
02H
FFH
01H
MM1
04H
04H
00H
00H
FFH
01H
01H
01H

[Figure 4-15](/x86/pshufb#fig-4-15). PSHUFB with 64-Bit Operands

## Intel C/C++ Compiler Intrinsic Equivalent

```
VPSHUFB __m512i _mm512_shuffle_epi8(__m512i a, __m512i b);

```

```
VPSHUFB __m512i _mm512_mask_shuffle_epi8(__m512i s, __mmask64 k, __m512i a, __m512i b);

```

```
VPSHUFB __m512i _mm512_maskz_shuffle_epi8( __mmask64 k, __m512i a, __m512i b);

```

```
VPSHUFB __m256i _mm256_mask_shuffle_epi8(__m256i s, __mmask32 k, __m256i a, __m256i b);

```

```
VPSHUFB __m256i _mm256_maskz_shuffle_epi8( __mmask32 k, __m256i a, __m256i b);

```

```
VPSHUFB __m128i _mm_mask_shuffle_epi8(__m128i s, __mmask16 k, __m128i a, __m128i b);

```

```
VPSHUFB __m128i _mm_maskz_shuffle_epi8( __mmask16 k, __m128i a, __m128i b);

```

```
PSHUFB: __m64 _mm_shuffle_pi8 (__m64 a, __m64 b)

```

```
(V)PSHUFB: __m128i _mm_shuffle_epi8 (__m128i a, __m128i b)

```

```
VPSHUFB:__m256i _mm256_shuffle_epi8(__m256i a, __m256i b)

```

## SIMD Floating-Point Exceptions

None.

## Other Exceptions

Non-EVEX-encoded instruction, see Table 2-21, “Type 4 Class Exception Conditions.”

EVEX-encoded instruction, see Exceptions Type E4NF.nb in Table 2-50, “Type E4NF Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
