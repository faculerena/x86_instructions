# PINSRW**Insert Word**

| Opcode/Instruction                                            | Op/ En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                                                     |
| ------------------------------------------------------------- | ------ | ---------------------- | ------------------ | --------------------------------------------------------------------------------------------------------------- |
| NP 0F C4 /_r_ ib1 PINSRW mm, r32/m16, imm8                    | A      | V/V                    | SSE                | Insert the low word from r32 or from m16 into mm at the word position specified by imm8.                        |
| 66 0F C4 /_r_ ib PINSRW xmm, r32/m16, imm8                    | A      | V/V                    | SSE2               | Move the low word of r32 or from m16 into xmm at the word position specified by imm8.                           |
| VEX.128.66.0F.W0 C4 /r ib VPINSRW xmm1, xmm2, r32/m16, imm8   | B      | V2/V                   | AVX                | Insert the word from r32/m16 at the offset indicated by imm8 into the value from xmm2 and store result in xmm1. |
| EVEX.128.66.0F.WIG C4 /r ib VPINSRW xmm1, xmm2, r32/m16, imm8 | C      | V/V                    | AVX512BW           | Insert the word from r32/m16 at the offset indicated by imm8 into the value from xmm2 and store result in xmm1. |

> 1. See note in Section 2.5, “Intel® AVX and Intel® SSE Instruction Exception Classification,” in the Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 2A, and Section 23.25.3, “Exception Conditions of Legacy SIMD Instructions Operating on MMX Registers,” in the Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 3B.
> 2. In 64-bit mode, VEX.W1 is ignored for VPINSRW (similar to legacy REX.W=1 prefix in PINSRW).

## Instruction Operand Encoding

| Op/En | Tuple Type    | Operand 1     | Operand 2     | Operand 3     | Operand 4 |
| ----- | ------------- | ------------- | ------------- | ------------- | --------- |
| A     | N/A           | ModRM:reg (w) | ModRM:r/m (r) | imm8          | N/A       |
| B     | N/A           | ModRM:reg (w) | VEX.vvvv (r)  | ModRM:r/m (r) | imm8      |
| C     | Tuple1 Scalar | ModRM:reg (w) | EVEX.vvvv (r) | ModRM:r/m (r) | imm8      |

## Description

Three operand MMX and SSE instructions:

Copies a word from the source operand and inserts it in the destination operand at the location specified with the count operand. (The other words in the destination register are left untouched.) The source operand can be a general-purpose register or a 16-bit memory location. (When the source operand is a general-purpose register, the low word of the register is copied.) The destination operand can be an MMX technology register or an XMM register. The count operand is an 8-bit immediate. When specifying a word location in an MMX technology register, the 2 least-significant bits of the count operand specify the location; for an XMM register, the 3 least-significant bits specify the location.

Bits (MAXVL-1:128) of the corresponding YMM destination register remain unchanged.

Four operand AVX and AVX-512 instructions:

Combines a word from the first source operand with the second source operand, and inserts it in the destination operand at the location specified with the count operand. The second source operand can be a general-purpose register or a 16-bit memory location. (When the source operand is a general-purpose register, the low word of the register is copied.) The first source and destination operands are XMM registers. The count operand is an 8-bit immediate. When specifying a word location, the 3 least-significant bits specify the location.

Bits (MAXVL-1:128) of the destination YMM register are zeroed. VEX.L/EVEX.L’L must be 0, otherwise the instruction will #​​​UD.

## Operation

### PINSRW dest, src, imm8 (MMX)

```
SEL := imm8[1:0]
DEST.word[SEL] := src.word[0]

```

### PINSRW dest, src, imm8 (SSE)

```
SEL := imm8[2:0]
DEST.word[SEL] := src.word[0]

```

### VPINSRW dest, src1, src2, imm8 (AVX/AVX512)

```
SEL := imm8[2:0]
DEST := src1
DEST.word[SEL] := src2.word[0]
DEST[MAXVL-1:128] := 0

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
PINSRW __m64 _mm_insert_pi16 (__m64 a, int d, int n)

```

```
PINSRW __m128i _mm_insert_epi16 ( __m128i a, int b, int imm)

```

## Flags Affected

None.

## Numeric Exceptions

None.

## Other Exceptions

EVEX-encoded instruction, see Table 2-22, “Type 5 Class Exception Conditions.”

EVEX-encoded instruction, see Table 2-57, “Type E9NF Class Exception Conditions.”

Additionally:

| #​​​UD | If VEX.L = 1 or EVEX.L’L > 0. |
| ------ | ----------------------------- |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
