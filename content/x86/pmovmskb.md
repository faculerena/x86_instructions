# PMOVMSKB**Move Byte Mask**

| Opcode/Instruction                          | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                          |
| ------------------------------------------- | ----- | ---------------------- | ------------------ | ------------------------------------------------------------------------------------ |
| NP 0F D7 /r1 PMOVMSKB reg, mm               | RM    | V/V                    | SSE                | Move a byte mask of mm to reg. The upper bits of r32 or r64 are zeroed               |
| 66 0F D7 /r PMOVMSKB reg, xmm               | RM    | V/V                    | SSE2               | Move a byte mask of xmm to reg. The upper bits of r32 or r64 are zeroed              |
| VEX.128.66.0F.WIG D7 /r VPMOVMSKB reg, xmm1 | RM    | V/V                    | AVX                | Move a byte mask of xmm1 to reg. The upper bits of r32 or r64 are filled with zeros. |
| VEX.256.66.0F.WIG D7 /r VPMOVMSKB reg, ymm1 | RM    | V/V                    | AVX2               | Move a 32-bit mask of ymm1 to reg. The upper bits of r64 are filled with zeros.      |

> 1. See note in Section 2.5, “Intel® AVX and Intel® SSE Instruction Exception Classification,” in the Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 2A, and Section 23.25.3, “Exception Conditions of Legacy SIMD Instructions Operating on MMX Registers,” in the Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 3B.

## Instruction Operand Encoding

| Op/En | Operand 1     | Operand 2     | Operand 3 | Operand 4 |
| ----- | ------------- | ------------- | --------- | --------- |
| RM    | ModRM:reg (w) | ModRM:r/m (r) | N/A       | N/A       |

## Description

Creates a mask made up of the most significant bit of each byte of the source operand (second operand) and stores the result in the low byte or word of the destination operand (first operand).

The byte mask is 8 bits for 64-bit source operand, 16 bits for 128-bit source operand and 32 bits for 256-bit source operand. The destination operand is a general-purpose register.

In 64-bit mode, the instruction can access additional registers (XMM8-XMM15, R8-R15) when used with a REX.R prefix. The default operand size is 64-bit in 64-bit mode.

Legacy SSE version: The source operand is an MMX technology register.

128-bit Legacy SSE version: The source operand is an XMM register.

VEX.128 encoded version: The source operand is an XMM register.

VEX.256 encoded version: The source operand is a YMM register.

Note: VEX.vvvv is reserved and must be 1111b.

## Operation

### PMOVMSKB (With 64-bit Source Operand and r32)

```
r32[0] := SRC[7];
r32[1] := SRC[15];
(* Repeat operation for bytes 2 through 6 *)
r32[7] := SRC[63];
r32[31:8] := ZERO_FILL;

```

### (V)PMOVMSKB (With 128-bit Source Operand and r32)

```
r32[0] := SRC[7];
r32[1] := SRC[15];
(* Repeat operation for bytes 2 through 14 *)
r32[15] := SRC[127];
r32[31:16] := ZERO_FILL;

```

### VPMOVMSKB (With 256-bit Source Operand and r32)

```
r32[0] := SRC[7];
r32[1] := SRC[15];
(* Repeat operation for bytes 3rd through 31*)
r32[31] := SRC[255];

```

### PMOVMSKB (With 64-bit Source Operand and r64)

```
r64[0] := SRC[7];
r64[1] := SRC[15];
(* Repeat operation for bytes 2 through 6 *)
r64[7] := SRC[63];
r64[63:8] := ZERO_FILL;

```

### (V)PMOVMSKB (With 128-bit Source Operand and r64)

```
r64[0] := SRC[7];
r64[1] := SRC[15];
(* Repeat operation for bytes 2 through 14 *)
r64[15] := SRC[127];
r64[63:16] := ZERO_FILL;

```

### VPMOVMSKB (With 256-bit Source Operand and r64)

```
r64[0] := SRC[7];
r64[1] := SRC[15];
(* Repeat operation for bytes 2 through 31*)
r64[31] := SRC[255];
r64[63:32] := ZERO_FILL;

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
PMOVMSKB int _mm_movemask_pi8(__m64 a)

```

```
(V)PMOVMSKB int _mm_movemask_epi8 ( __m128i a)

```

```
VPMOVMSKB int _mm256_movemask_epi8 ( __m256i a)

```

## Flags Affected

None.

## Numeric Exceptions

None.

## Other Exceptions

See Table 2-24, “Type 7 Class Exception Conditions,” additionally:

| #​​​UD | If VEX.vvvv ≠ 1111B. |
| ------ | -------------------- |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
