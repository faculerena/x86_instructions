#MOVSS
**Move or Merge Scalar Single Precision Floating**

| Opcode/Instruction                                       | Op / En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                                               |
| -------------------------------------------------------- | ------- | ---------------------- | ------------------ | --------------------------------------------------------------------------------------------------------- |
| F3 0F 10 /r MOVSS xmm1, xmm2                             | A       | V/V                    | SSE                | Merge scalar single precision floating-point value from xmm2 to xmm1 register.                            |
| F3 0F 10 /r MOVSS xmm1, m32                              | A       | V/V                    | SSE                | Load scalar single precision floating-point value from m32 to xmm1 register.                              |
| VEX.LIG.F3.0F.WIG 10 /r VMOVSS xmm1, xmm2, xmm3          | B       | V/V                    | AVX                | Merge scalar single precision floating-point value from xmm2 and xmm3 to xmm1 register                    |
| VEX.LIG.F3.0F.WIG 10 /r VMOVSS xmm1, m32                 | D       | V/V                    | AVX                | Load scalar single precision floating-point value from m32 to xmm1 register.                              |
| F3 0F 11 /r MOVSS xmm2/m32, xmm1                         | C       | V/V                    | SSE                | Move scalar single precision floating-point value from xmm1 register to xmm2/m32.                         |
| VEX.LIG.F3.0F.WIG 11 /r VMOVSS xmm1, xmm2, xmm3          | E       | V/V                    | AVX                | Move scalar single precision floating-point value from xmm2 and xmm3 to xmm1 register.                    |
| VEX.LIG.F3.0F.WIG 11 /r VMOVSS m32, xmm1                 | C       | V/V                    | AVX                | Move scalar single precision floating-point value from xmm1 register to m32.                              |
| EVEX.LLIG.F3.0F.W0 10 /r VMOVSS xmm1 {k1}{z}, xmm2, xmm3 | B       | V/V                    | AVX512F            | Move scalar single precision floating-point value from xmm2 and xmm3 to xmm1 register under writemask k1. |
| EVEX.LLIG.F3.0F.W0 10 /r VMOVSS xmm1 {k1}{z}, m32        | F       | V/V                    | AVX512F            | Move scalar single precision floating-point values from m32 to xmm1 under writemask k1.                   |
| EVEX.LLIG.F3.0F.W0 11 /r VMOVSS xmm1 {k1}{z}, xmm2, xmm3 | E       | V/V                    | AVX512F            | Move scalar single precision floating-point value from xmm2 and xmm3 to xmm1 register under writemask k1. |
| EVEX.LLIG.F3.0F.W0 11 /r VMOVSS m32 {k1}, xmm1           | G       | V/V                    | AVX512F            | Move scalar single precision floating-point values from xmm1 to m32 under writemask k1.                   |

## Instruction Operand Encoding

| Op/En | Tuple Type    | Operand 1        | Operand 2     | Operand 3     | Operand 4 |
| ----- | ------------- | ---------------- | ------------- | ------------- | --------- |
| A     | N/A           | ModRM:reg (r, w) | ModRM:r/m (r) | N/A           | N/A       |
| B     | N/A           | ModRM:reg (w)    | VEX.vvvv (r)  | ModRM:r/m (r) | N/A       |
| C     | N/A           | ModRM:r/m (w)    | ModRM:reg (r) | N/A           | N/A       |
| D     | N/A           | ModRM:reg (w)    | ModRM:r/m (r) | N/A           | N/A       |
| E     | N/A           | ModRM:r/m (w)    | EVEX.vvvv (r) | ModRM:reg (r) | N/A       |
| F     | Tuple1 Scalar | ModRM:reg (r, w) | ModRM:r/m (r) | N/A           | N/A       |
| G     | Tuple1 Scalar | ModRM:r/m (w)    | ModRM:reg (r) | N/A           | N/A       |

## Description

Moves a scalar single precision floating-point value from the source operand (second operand) to the destination operand (first operand). The source and destination operands can be XMM registers or 32-bit memory locations. This instruction can be used to move a single precision floating-point value to and from the low doubleword of an XMM register and a 32-bit memory location, or to move a single precision floating-point value between the low doublewords of two XMM registers. The instruction cannot be used to transfer data between memory locations.

Legacy version: When the source and destination operands are XMM registers, bits (MAXVL-1:32) of the corresponding destination register are unmodified. When the source operand is a memory location and destination

operand is an XMM registers, Bits (127:32) of the destination operand is cleared to all 0s, bits MAXVL:128 of the destination operand remains unchanged.

VEX and EVEX encoded register-register syntax: Moves a scalar single precision floating-point value from the second source operand (the third operand) to the low doubleword element of the destination operand (the first operand). Bits 127:32 of the destination operand are copied from the first source operand (the second operand). Bits (MAXVL-1:128) of the corresponding destination register are zeroed.

VEX and EVEX encoded memory load syntax: When the source operand is a memory location and destination operand is an XMM registers, bits MAXVL:32 of the destination operand is cleared to all 0s.

EVEX encoded versions: The low doubleword of the destination is updated according to the writemask.

Note: For memory store form instruction “VMOVSS m32, xmm1”, VEX.vvvv is reserved and must be 1111b otherwise instruction will #​​​UD. For memory store form instruction “VMOVSS mv {k1}, xmm1”, EVEX.vvvv is reserved and must be 1111b otherwise instruction will #​​​UD.

Software should ensure VMOVSS is encoded with VEX.L=0. Encoding VMOVSS with VEX.L=1 may encounter unpredictable behavior across different processor generations.

## Operation

### VMOVSS (EVEX.LLIG.F3.0F.W0 11 /r When the Source Operand is Memory and the Destination is an XMM Register)

```
IF k1[0] or *no writemask*
    THEN DEST[31:0] := SRC[31:0]
    ELSE
        IF *merging-masking* ; merging-masking
            THEN *DEST[31:0] remains unchanged*
            ELSE ; zeroing-masking
                THEN DEST[31:0] := 0
        FI;
FI;
DEST[MAXVL-1:32] := 0

```

### VMOVSS (EVEX.LLIG.F3.0F.W0 10 /r When the Source Operand is an XMM Register and the Destination is Memory)

```
IF k1[0] or *no writemask*
    THEN DEST[31:0] := SRC[31:0]
    ELSE *DEST[31:0] remains unchanged* ; merging-masking
FI;

```

### VMOVSS (EVEX.LLIG.F3.0F.W0 10/11 /r Where the Source and Destination are XMM Registers)

```
IF k1[0] or *no writemask*
    THEN DEST[31:0] := SRC2[31:0]
    ELSE
        IF *merging-masking* ; merging-masking
            THEN *DEST[31:0] remains unchanged*
            ELSE ; zeroing-masking
                THEN DEST[31:0] := 0
        FI;
FI;
DEST[127:32] := SRC1[127:32]
DEST[MAXVL-1:128] := 0

```

### MOVSS (Legacy SSE Version When the Source and Destination Operands are Both XMM Registers)

```
DEST[31:0] := SRC[31:0]
DEST[MAXVL-1:32] (Unmodified)

```

### VMOVSS (VEX.128.F3.0F 11 /r Where the Destination is an XMM Register)

```
DEST[31:0] := SRC2[31:0]
DEST[127:32] := SRC1[127:32]
DEST[MAXVL-1:128] := 0

```

### VMOVSS (VEX.128.F3.0F 10 /r Where the Source and Destination are XMM Registers)

```
DEST[31:0] := SRC2[31:0]
DEST[127:32] := SRC1[127:32]
DEST[MAXVL-1:128] := 0

```

### VMOVSS (VEX.128.F3.0F 10 /r When the Source Operand is Memory and the Destination is an XMM Register)

```
DEST[31:0] := SRC[31:0]
DEST[MAXVL-1:32] := 0

```

### MOVSS/VMOVSS (When the Source Operand is an XMM Register and the Destination is Memory)

```
DEST[31:0] := SRC[31:0]

```

### MOVSS (Legacy SSE Version when the Source Operand is Memory and the Destination is an XMM Register)

```
DEST[31:0] := SRC[31:0]
DEST[127:32] := 0
DEST[MAXVL-1:128] (Unmodified)

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
VMOVSS __m128 _mm_mask_load_ss(__m128 s, __mmask8 k, float * p);

```

```
VMOVSS __m128 _mm_maskz_load_ss( __mmask8 k, float * p);

```

```
VMOVSS __m128 _mm_mask_move_ss(__m128 sh, __mmask8 k, __m128 sl, __m128 a);

```

```
VMOVSS __m128 _mm_maskz_move_ss( __mmask8 k, __m128 s, __m128 a);

```

```
VMOVSS void _mm_mask_store_ss(float * p, __mmask8 k, __m128 a);

```

```
MOVSS __m128 _mm_load_ss(float * p)

```

```
MOVSS void_mm_store_ss(float * p, __m128 a)

```

```
MOVSS __m128 _mm_move_ss(__m128 a, __m128 b)

```

## SIMD Floating-Point Exceptions

None.

## Other Exceptions

Non-EVEX-encoded instruction, see Table 2-22, “Type 5 Class Exception Conditions,” additionally:

| #​​​UD | If VEX.vvvv != 1111B. |
| ------ | --------------------- |

EVEX-encoded instruction, see Table 2-58, “Type E10 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
