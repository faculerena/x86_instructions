#MOVLHPS
**Move Packed Single Precision Floating**

| Opcode/Instruction                             | Op / En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                                                 |
| ---------------------------------------------- | ------- | ---------------------- | ------------------ | ----------------------------------------------------------------------------------------------------------- |
| NP 0F 16 /r MOVLHPS xmm1, xmm2                 | RM      | V/V                    | SSE                | Move two packed single precision floating-point values from low quadword of xmm2 to high quadword of xmm1.  |
| VEX.128.0F.WIG 16 /r VMOVLHPS xmm1, xmm2, xmm3 | RVM     | V/V                    | AVX                | Merge two packed single precision floating-point values from low quadword of xmm3 and low quadword of xmm2. |
| EVEX.128.0F.W0 16 /r VMOVLHPS xmm1, xmm2, xmm3 | RVM     | V/V                    | AVX512F            | Merge two packed single precision floating-point values from low quadword of xmm3 and low quadword of xmm2. |

## Instruction Operand Encoding1

> 1. ModRM.MOD = 011B required

| Op/En | Operand 1     | Operand 2                    | Operand 3     | Operand 4 |
| ----- | ------------- | ---------------------------- | ------------- | --------- |
| RM    | ModRM:reg (w) | ModRM:r/m (r)                | N/A           | N/A       |
| RVM   | ModRM:reg (w) | VEX.vvvv (r) / EVEX.vvvv (r) | ModRM:r/m (r) | N/A       |

## Description

This instruction cannot be used for memory to register moves.

128-bit two-argument form:

Moves two packed single precision floating-point values from the low quadword of the second XMM argument (second operand) to the high quadword of the first XMM register (first argument). The low quadword of the destination operand is left unchanged. Bits (MAXVL-1:128) of the corresponding destination register are unmodified.

128-bit three-argument forms:

Moves two packed single precision floating-point values from the low quadword of the third XMM argument (third operand) to the high quadword of the destination (first operand). Copies the low quadword from the second XMM argument (second operand) to the low quadword of the destination (first operand). Bits (MAXVL-1:128) of the corresponding destination register are zeroed.

If VMOVLHPS is encoded with VEX.L or EVEX.L’L= 1, an attempt to execute the instruction encoded with VEX.L or EVEX.L’L= 1 will cause an #​​​UD exception.

## Operation

### MOVLHPS (128-bit Two-Argument Form)

```
DEST[63:0] (Unmodified)
DEST[127:64] := SRC[63:0]
DEST[MAXVL-1:128] (Unmodified)

```

### VMOVLHPS (128-bit Three-Argument Form - VEX & EVEX)

```
DEST[63:0] := SRC1[63:0]
DEST[127:64] := SRC2[63:0]
DEST[MAXVL-1:128] := 0

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
MOVLHPS __m128 _mm_movelh_ps(__m128 a, __m128 b)

```

## SIMD Floating-Point Exceptions

None.

## Other Exceptions

Non-EVEX-encoded instruction, see Table 2-24, “Type 7 Class Exception Conditions,” additionally:

| #​​​UD | If VEX.L = 1. |
| ------ | ------------- |

EVEX-encoded instruction, see Exceptions Type E7NM.128 in Table 2-55, “Type E7NM Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
