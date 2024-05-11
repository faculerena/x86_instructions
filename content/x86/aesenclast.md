#AESENCLAST
**Perform Last Round of an AES Encryption Flow**

| Opcode/Instruction                                           | Op/En | 64/32-bit Mode | CPUID Feature Flag | Description                                                                                                                                                        |
| ------------------------------------------------------------ | ----- | -------------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 66 0F 38 DD /r AESENCLAST xmm1, xmm2/m128                    | A     | V/V            | AES                | Perform the last round of an AES encryption flow, using one 128-bit data (state) from xmm1 with one 128-bit round key from xmm2/m128.                              |
| VEX.128.66.0F38.WIG DD /r VAESENCLAST xmm1, xmm2, xmm3/m128  | B     | V/V            | AES AVX            | Perform the last round of an AES encryption flow, using one 128-bit data (state) from xmm2 with one 128-bit round key from xmm3/m128; store the result in xmm1.    |
| VEX.256.66.0F38.WIG DD /r VAESENCLAST ymm1, ymm2, ymm3/m256  | B     | V/V            | VAES               | Perform the last round of an AES encryption flow, using two 128-bit data (state) from ymm2 with two 128-bit round keys from ymm3/m256; store the result in ymm1.   |
| EVEX.128.66.0F38.WIG DD /r VAESENCLAST xmm1, xmm2, xmm3/m128 | C     | V/V            | VAES AVX512VL      | Perform the last round of an AES encryption flow, using one 128-bit data (state) from xmm2 with one 128-bit round key from xmm3/m128; store the result in xmm1.    |
| EVEX.256.66.0F38.WIG DD /r VAESENCLAST ymm1, ymm2, ymm3/m256 | C     | V/V            | VAES AVX512VL      | Perform the last round of an AES encryption flow, using two 128-bit data (state) from ymm2 with two 128-bit round keys from ymm3/m256; store the result in ymm1.   |
| EVEX.512.66.0F38.WIG DD /r VAESENCLAST zmm1, zmm2, zmm3/m512 | C     | V/V            | VAES AVX512F       | Perform the last round of an AES encryption flow, using four 128-bit data (state) from zmm2 with four 128-bit round keys from zmm3/m512; store the result in zmm1. |

## Instruction Operand Encoding

| Op/En | Tuple    | Operand 1        | Operand 2     | Operand 3     | Operand 4 |
| ----- | -------- | ---------------- | ------------- | ------------- | --------- |
| A     | N/A      | ModRM:reg (r, w) | ModRM:r/m (r) | N/A           | N/A       |
| B     | N/A      | ModRM:reg (w)    | VEX.vvvv (r)  | ModRM:r/m (r) | N/A       |
| C     | Full Mem | ModRM:reg (w)    | EVEX.vvvv (r) | ModRM:r/m (r) | N/A       |

## Description

This instruction performs the last round of an AES encryption flow using one/two/four (depending on vector length) 128-bit data (state) from the first source operand with one/two/four (depending on vector length) round key(s) from the second source operand, and stores the result in the destination operand.

VEX and EVEX encoded versions of the instruction allows 3-operand (non-destructive) operation. The legacy encoded versions of the instruction require that the first source operand and the destination operand are the same and must be an XMM register.

The EVEX encoded form of this instruction does not support memory fault suppression.

## Operation

### AESENCLAST

```
STATE := SRC1;
RoundKey := SRC2;
STATE := ShiftRows( STATE );
STATE := SubBytes( STATE );
DEST[127:0] := STATE XOR RoundKey;
DEST[MAXVL-1:128] (Unmodified)

```

### VAESENCLAST (128b and 256b VEX Encoded Versions)

```
(KL, VL) = (1,128), (2,256)
FOR I=0 to KL-1:
    STATE := SRC1.xmm[i]
    RoundKey := SRC2.xmm[i]
    STATE := ShiftRows( STATE )
    STATE := SubBytes( STATE )
    DEST.xmm[i] := STATE XOR RoundKey
DEST[MAXVL-1:VL] := 0

```

### VAESENCLAST (EVEX Encoded Version)

```
(KL,VL) = (1,128), (2,256), (4,512)
FOR i = 0 to KL-1:
    STATE := SRC1.xmm[i]
    RoundKey := SRC2.xmm[i]
    STATE := ShiftRows( STATE )
    STATE := SubBytes( STATE )
    DEST.xmm[i] := STATE XOR RoundKey
DEST[MAXVL-1:VL] := 0

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
(V)AESENCLAST __m128i _mm_aesenclast (__m128i, __m128i)

```

```
VAESENCLAST __m256i _mm256_aesenclast_epi128(__m256i, __m256i);

```

```
VAESENCLAST __m512i _mm512_aesenclast_epi128(__m512i, __m512i);

```

## SIMD Floating-Point Exceptions

None.

## Other Exceptions

See Table 2-21, “Type 4 Class Exception Conditions.”

EVEX-encoded: See Table 2-50, “Type E4NF Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
