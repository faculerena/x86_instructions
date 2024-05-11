#PCLMULQDQ
**Carry**

| Opcode/Instruction                                                    | Op/En | 64/32 bit Mode Support | CPUID Feature Flag  | Description                                                                                                                                                                                                 |
| --------------------------------------------------------------------- | ----- | ---------------------- | ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 66 0F 3A 44 /r ib PCLMULQDQ xmm1, xmm2/m128, imm8                     | A     | V/V                    | PCLMULQDQ           | Carry-less multiplication of one quadword of xmm1 by one quadword of xmm2/m128, stores the 128-bit result in xmm1. The immediate is used to determine which quadwords of xmm1 and xmm2/m128 should be used. |
| VEX.128.66.0F3A.WIG 44 /r ib VPCLMULQDQ xmm1, xmm2, xmm3/m128, imm8   | B     | V/V                    | PCLMULQDQ AVX       | Carry-less multiplication of one quadword of xmm2 by one quadword of xmm3/m128, stores the 128-bit result in xmm1. The immediate is used to determine which quadwords of xmm2 and xmm3/m128 should be used. |
| VEX.256.66.0F3A.WIG 44 /r /ib VPCLMULQDQ ymm1, ymm2, ymm3/m256, imm8  | B     | V/V                    | VPCLMULQDQ AVX      | Carry-less multiplication of one quadword of ymm2 by one quadword of ymm3/m256, stores the 128-bit result in ymm1. The immediate is used to determine which quadwords of ymm2 and ymm3/m256 should be used. |
| EVEX.128.66.0F3A.WIG 44 /r /ib VPCLMULQDQ xmm1, xmm2, xmm3/m128, imm8 | C     | V/V                    | VPCLMULQDQ AVX512VL | Carry-less multiplication of one quadword of xmm2 by one quadword of xmm3/m128, stores the 128-bit result in xmm1. The immediate is used to determine which quadwords of xmm2 and xmm3/m128 should be used. |
| EVEX.256.66.0F3A.WIG 44 /r /ib VPCLMULQDQ ymm1, ymm2, ymm3/m256, imm8 | C     | V/V                    | VPCLMULQDQ AVX512VL | Carry-less multiplication of one quadword of ymm2 by one quadword of ymm3/m256, stores the 128-bit result in ymm1. The immediate is used to determine which quadwords of ymm2 and ymm3/m256 should be used. |
| EVEX.512.66.0F3A.WIG 44 /r /ib VPCLMULQDQ zmm1, zmm2, zmm3/m512, imm8 | C     | V/V                    | VPCLMULQDQ AVX512F  | Carry-less multiplication of one quadword of zmm2 by one quadword of zmm3/m512, stores the 128-bit result in zmm1. The immediate is used to determine which quadwords of zmm2 and zmm3/m512 should be used. |

## Instruction Operand Encoding

| Op/En | Tuple    | Operand 1        | Operand 2     | Operand 3     | Operand 4 |
| ----- | -------- | ---------------- | ------------- | ------------- | --------- |
| A     | N/A      | ModRM:reg (r, w) | ModRM:r/m (r) | imm8          | N/A       |
| B     | N/A      | ModRM:reg (w)    | VEX.vvvv (r)  | ModRM:r/m (r) | imm8      |
| C     | Full Mem | ModRM:reg (w)    | EVEX.vvvv (r) | ModRM:r/m (r) | imm8 (r)  |

## Description

Performs a carry-less multiplication of two quadwords, selected from the first source and second source operand according to the value of the immediate byte. Bits 4 and 0 are used to select which 64-bit half of each operand to use according to [Table 4-13](/x86/pclmulqdq#tbl-4-13), other bits of the immediate byte are ignored.

The EVEX encoded form of this instruction does not support memory fault suppression.

| Imm[4] | Imm[0] | PCLMULQDQ Operation                  |
| ------ | ------ | ------------------------------------ |
| 0      | 0      | CL_MUL( SRC21[63:0], SRC1[63:0] )    |
| 0      | 1      | CL_MUL( SRC2[63:0], SRC1[127:64] )   |
| 1      | 0      | CL_MUL( SRC2[127:64], SRC1[63:0] )   |
| 1      | 1      | CL_MUL( SRC2[127:64], SRC1[127:64] ) |

[Table 4-13](/x86/pclmulqdq#tbl-4-13). PCLMULQDQ Quadword Selection of Immediate Byte

> 1. SRC2 denotes the second source operand, which can be a register or memory; SRC1 denotes the first source and destination operand.

The first source operand and the destination operand are the same and must be a ZMM/YMM/XMM register. The second source operand can be a ZMM/YMM/XMM register or a 512/256/128-bit memory location. Bits (VL_MAX-1:128) of the corresponding YMM destination register remain unchanged.

Compilers and assemblers may implement the following pseudo-op syntax to simplify programming and emit the required encoding for imm8.

| Pseudo-Op                   | Imm8 Encoding  |
| --------------------------- | -------------- |
| **PCLMULLQLQDQ** xmm1, xmm2 | **0000_0000B** |
| **PCLMULHQLQDQ** xmm1, xmm2 | **0000_0001B** |
| **PCLMULLQHQDQ** xmm1, xmm2 | **0001_0000B** |
| **PCLMULHQHQDQ** xmm1, xmm2 | **0001_0001B** |

[Table 4-14](/x86/pclmulqdq#tbl-4-14). Pseudo-Op and PCLMULQDQ Implementation

## Operation

```
define PCLMUL128(X,Y): // helper function
    FOR i := 0 to 63:
        TMP [ i ] := X[ 0 ] and Y[ i ]
        FOR j := 1 to i:
            TMP [ i ] := TMP [ i ] xor (X[ j ] and Y[ i - j ])
        DEST[ i ] := TMP[ i ]
    FOR i := 64 to 126:
        TMP [ i ] := 0
        FOR j := i - 63 to 63:
            TMP [ i ] := TMP [ i ] xor (X[ j ] and Y[ i - j ])
        DEST[ i ] := TMP[ i ]
    DEST[127] := 0;
    RETURN DEST // 128b vector

```

### PCLMULQDQ (SSE Version)

```
IF imm8[0] = 0:
    TEMP1 := SRC1.qword[0]
ELSE:
    TEMP1 := SRC1.qword[1]
IF imm8[4] = 0:
    TEMP2 := SRC2.qword[0]
ELSE:
    TEMP2 := SRC2.qword[1]
DEST[127:0] := PCLMUL128(TEMP1, TEMP2)
DEST[MAXVL-1:128] (Unmodified)

```

### VPCLMULQDQ (128b and 256b VEX Encoded Versions)

```
(KL,VL) = (1,128), (2,256)
FOR i= 0 to KL-1:
    IF imm8[0] = 0:
        TEMP1 := SRC1.xmm[i].qword[0]
    ELSE:
        TEMP1 := SRC1.xmm[i].qword[1]
    IF imm8[4] = 0:
        TEMP2 := SRC2.xmm[i].qword[0]
    ELSE:
        TEMP2 := SRC2.xmm[i].qword[1]
    DEST.xmm[i] := PCLMUL128(TEMP1, TEMP2)
DEST[MAXVL-1:VL] := 0

```

### VPCLMULQDQ (EVEX Encoded Version)

```
(KL,VL) = (1,128), (2,256), (4,512)
FOR i = 0 to KL-1:
    IF imm8[0] = 0:
        TEMP1 := SRC1.xmm[i].qword[0]
    ELSE:
        TEMP1 := SRC1.xmm[i].qword[1]
    IF imm8[4] = 0:
        TEMP2 := SRC2.xmm[i].qword[0]
    ELSE:
        TEMP2 := SRC2.xmm[i].qword[1]
    DEST.xmm[i] := PCLMUL128(TEMP1, TEMP2)
DEST[MAXVL-1:VL] := 0

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
(V)PCLMULQDQ __m128i _mm_clmulepi64_si128 (__m128i, __m128i, const int)

```

```
VPCLMULQDQ __m256i _mm256_clmulepi64_epi128(__m256i, __m256i, const int);

```

```
VPCLMULQDQ __m512i _mm512_clmulepi64_epi128(__m512i, __m512i, const int);

```

## SIMD Floating-Point Exceptions

None.

## Other Exceptions

See Table 2-21, “Type 4 Class Exception Conditions,” additionally:

| #​​​UD | If VEX.L = 1. |
| ------ | ------------- |

EVEX-encoded: See Table 2-50, “Type E4NF Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
