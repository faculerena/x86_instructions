#MOVHPS
**Move High Packed Single Precision Floating**

| Opcode/Instruction                           | Op / En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                                    |
| -------------------------------------------- | ------- | ---------------------- | ------------------ | ---------------------------------------------------------------------------------------------- |
| NP 0F 16 /r MOVHPS xmm1, m64                 | A       | V/V                    | SSE                | Move two packed single precision floating-point values from m64 to high quadword of xmm1.      |
| VEX.128.0F.WIG 16 /r VMOVHPS xmm2, xmm1, m64 | B       | V/V                    | AVX                | Merge two packed single precision floating-point values from m64 and the low quadword of xmm1. |
| EVEX.128.0F.W0 16 /r VMOVHPS xmm2, xmm1, m64 | D       | V/V                    | AVX512F            | Merge two packed single precision floating-point values from m64 and the low quadword of xmm1. |
| NP 0F 17 /r MOVHPS m64, xmm1                 | C       | V/V                    | SSE                | Move two packed single precision floating-point values from high quadword of xmm1 to m64.      |
| VEX.128.0F.WIG 17 /r VMOVHPS m64, xmm1       | C       | V/V                    | AVX                | Move two packed single precision floating-point values from high quadword of xmm1 to m64.      |
| EVEX.128.0F.W0 17 /r VMOVHPS m64, xmm1       | E       | V/V                    | AVX512F            | Move two packed single precision floating-point values from high quadword of xmm1 to m64.      |

## Instruction Operand Encoding

| Op/En | Tuple Type | Operand 1        | Operand 2     | Operand 3     | Operand 4 |
| ----- | ---------- | ---------------- | ------------- | ------------- | --------- |
| A     | N/A        | ModRM:reg (r, w) | ModRM:r/m (r) | N/A           | N/A       |
| B     | N/A        | ModRM:reg (w)    | VEX.vvvv (r)  | ModRM:r/m (r) | N/A       |
| C     | N/A        | ModRM:r/m (w)    | ModRM:reg (r) | N/A           | N/A       |
| D     | Tuple2     | ModRM:reg (w)    | EVEX.vvvv (r) | ModRM:r/m (r) | N/A       |
| E     | Tuple2     | ModRM:r/m (w)    | ModRM:reg (r) | N/A           | N/A       |

## Description

This instruction cannot be used for register to register or memory to memory moves.

128-bit Legacy SSE load:

Moves two packed single precision floating-point values from the source 64-bit memory operand and stores them in the high 64-bits of the destination XMM register. The lower 64bits of the XMM register are preserved. Bits (MAXVL-1:128) of the corresponding destination register are preserved.

VEX.128 & EVEX encoded load:

Loads two single precision floating-point values from the source 64-bit memory operand (the third operand) and stores it in the upper 64-bits of the destination XMM register (first operand). The low 64-bits from the first source operand (the second operand) are copied to the lower 64-bits of the destination. Bits (MAXVL-1:128) of the corresponding destination register are zeroed.

128-bit store:

Stores two packed single precision floating-point values from the high 64-bits of the XMM register source (second operand) to the 64-bit memory location (first operand).

Note: VMOVHPS (store) (VEX.128.0F 17 /r) is legal and has the same behavior as the existing 0F 17 store. For VMOVHPS (store) VEX.vvvv and EVEX.vvvv are reserved and must be 1111b otherwise instruction will #​​​UD.

If VMOVHPS is encoded with VEX.L or EVEX.L’L= 1, an attempt to execute the instruction encoded with VEX.L or EVEX.L’L= 1 will cause an #​​​UD exception.

## Operation

### MOVHPS (128-bit Legacy SSE Load)

```
DEST[63:0] (Unmodified)
DEST[127:64] := SRC[63:0]
DEST[MAXVL-1:128] (Unmodified)

```

### VMOVHPS (VEX.128 and EVEX Encoded Load)

```
DEST[63:0] := SRC1[63:0]
DEST[127:64] := SRC2[63:0]
DEST[MAXVL-1:128] := 0

```

### VMOVHPS (Store)

```
DEST[63:0] := SRC[127:64]

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
MOVHPS __m128 _mm_loadh_pi ( __m128 a, __m64 *p)

```

```
MOVHPS void _mm_storeh_pi (__m64 *p, __m128 a)

```

## SIMD Floating-Point Exceptions

None.

## Other Exceptions

Non-EVEX-encoded instruction, see Table 2-22, “Type 5 Class Exception Conditions,” additionally:

| #​​​UD | If VEX.L = 1. |
| ------ | ------------- |

EVEX-encoded instruction, see Table 2-57, “Type E9NF Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
