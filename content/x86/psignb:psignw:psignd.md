# PSIGNB/PSIGNW/PSIGND**Packed SIGN**

| Opcode/Instruction                                      | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                                               |
| ------------------------------------------------------- | ----- | ---------------------- | ------------------ | --------------------------------------------------------------------------------------------------------- |
| NP 0F 38 08 /r1 PSIGNB mm1, mm2/m64                     | RM    | V/V                    | SSSE3              | Negate/zero/preserve packed byte integers in mm1 depending on the corresponding sign in mm2/m64.          |
| 66 0F 38 08 /r PSIGNB xmm1, xmm2/m128                   | RM    | V/V                    | SSSE3              | Negate/zero/preserve packed byte integers in xmm1 depending on the corresponding sign in xmm2/m128.       |
| NP 0F 38 09 /r1 PSIGNW mm1, mm2/m64                     | RM    | V/V                    | SSSE3              | Negate/zero/preserve packed word integers in mm1 depending on the corresponding sign in mm2/m128.         |
| 66 0F 38 09 /r PSIGNW xmm1, xmm2/m128                   | RM    | V/V                    | SSSE3              | Negate/zero/preserve packed word integers in xmm1 depending on the corresponding sign in xmm2/m128.       |
| NP 0F 38 0A /r1 PSIGND mm1, mm2/m64                     | RM    | V/V                    | SSSE3              | Negate/zero/preserve packed doubleword integers in mm1 depending on the corresponding sign in mm2/m128.   |
| 66 0F 38 0A /r PSIGND xmm1, xmm2/m128                   | RM    | V/V                    | SSSE3              | Negate/zero/preserve packed doubleword integers in xmm1 depending on the corresponding sign in xmm2/m128. |
| VEX.128.66.0F38.WIG 08 /r VPSIGNB xmm1, xmm2, xmm3/m128 | RVM   | V/V                    | AVX                | Negate/zero/preserve packed byte integers in xmm2 depending on the corresponding sign in xmm3/m128.       |
| VEX.128.66.0F38.WIG 09 /r VPSIGNW xmm1, xmm2, xmm3/m128 | RVM   | V/V                    | AVX                | Negate/zero/preserve packed word integers in xmm2 depending on the corresponding sign in xmm3/m128.       |
| VEX.128.66.0F38.WIG 0A /r VPSIGND xmm1, xmm2, xmm3/m128 | RVM   | V/V                    | AVX                | Negate/zero/preserve packed doubleword integers in xmm2 depending on the corresponding sign in xmm3/m128. |
| VEX.256.66.0F38.WIG 08 /r VPSIGNB ymm1, ymm2, ymm3/m256 | RVM   | V/V                    | AVX2               | Negate packed byte integers in ymm2 if the corresponding sign in ymm3/m256 is less than zero.             |
| VEX.256.66.0F38.WIG 09 /r VPSIGNW ymm1, ymm2, ymm3/m256 | RVM   | V/V                    | AVX2               | Negate packed 16-bit integers in ymm2 if the corresponding sign in ymm3/m256 is less than zero.           |
| VEX.256.66.0F38.WIG 0A /r VPSIGND ymm1, ymm2, ymm3/m256 | RVM   | V/V                    | AVX2               | Negate packed doubleword integers in ymm2 if the corresponding sign in ymm3/m256 is less than zero.       |

> 1. See note in Section 2.5, “Intel® AVX and Intel® SSE Instruction Exception Classification,” in the Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 2A, and Section 23.25.3, “Exception Conditions of Legacy SIMD Instructions Operating on MMX Registers,” in the Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 3B.

## Instruction Operand Encoding

| Op/En | Operand 1        | Operand 2     | Operand 3     | Operand 4 |
| ----- | ---------------- | ------------- | ------------- | --------- |
| RM    | ModRM:reg (r, w) | ModRM:r/m (r) | N/A           | N/A       |
| RVM   | ModRM:reg (w)    | VEX.vvvv (r)  | ModRM:r/m (r) | N/A       |

## Description

(V)PSIGNB/(V)PSIGNW/(V)PSIGND negates each data element of the destination operand (the first operand) if the signed integer value of the corresponding data element in the source operand (the second operand) is less than zero. If the signed integer value of a data element in the source operand is positive, the corresponding data element in the destination operand is unchanged. If a data element in the source operand is zero, the corresponding data element in the destination operand is set to zero.

(V)PSIGNB operates on signed bytes. (V)PSIGNW operates on 16-bit signed words. (V)PSIGND operates on signed 32-bit integers.

Legacy SSE instructions: Both operands can be MMX registers. In 64-bit mode, use the REX prefix to access additional registers.

128-bit Legacy SSE version: The first source and destination operands are XMM registers. The second source operand is an XMM register or a 128-bit memory location. Bits (MAXVL-1:128) of the corresponding YMM destination register remain unchanged.

VEX.128 encoded version: The first source and destination operands are XMM registers. The second source operand is an XMM register or a 128-bit memory location. Bits (MAXVL-1:128) of the destination YMM register are zeroed. VEX.L must be 0, otherwise instructions will #​​​UD.

VEX.256 encoded version: The first source and destination operands are YMM registers. The second source operand is an YMM register or a 256-bit memory location.

## Operation

```
def byte_sign(control, input_val):
    if control<0:
        return negate(input_val)
    elif control==0:
        return 0
    return input_val
def word_sign(control, input_val):
    if control<0:
        return negate(input_val)
    elif control==0:
        return 0
    return input_val
def dword_sign(control, input_val):
    if control<0:
        return negate(input_val)
    elif control==0:
        return 0
    return input_val

```

### PSIGNB srcdest, src

### // MMX 64-bit Operands

```
VL=64
KL := VL/8
for i in 0...KL-1:
    srcdest.byte[i] := byte_sign(src.byte[i], srcdest.byte[i])

```

### PSIGNW srcdest, src // MMX 64-bit Operands

```
VL=64
KL := VL/16
FOR i in 0...KL-1:
    srcdest.word[i] := word_sign(src.word[i], srcdest.word[i])

```

### PSIGND srcdest, src // MMX 64-bit Operands

```
VL=64
KL := VL/32
FOR i in 0...KL-1:
    srcdest.dword[i] := dword_sign(src.dword[i], srcdest.dword[i])

```

### PSIGNB srcdest, src // SSE 128-bit Operands

```
VL=128
KL := VL/8
FOR i in 0...KL-1:
    srcdest.byte[i] := byte_sign(src.byte[i], srcdest.byte[i])

```

### PSIGNW srcdest, src // SSE 128-bit Operands

```
VL=128
KL := VL/16
FOR i in 0...KL-1:
    srcdest.word[i] := word_sign(src.word[i], srcdest.word[i])

```

### PSIGND srcdest, src // SSE 128-bit Operands

```
VL=128
KL := VL/32
FOR i in 0...KL-1:
    srcdest.dword[i] := dword_sign(src.dword[i], srcdest.dword[i])

```

### VPSIGNB dest, src1, src2 // AVX 128-bit or 256-bit Operands

```
VL=(128,256)
KL := VL/8
FOR i in 0...KL-1:
    dest.byte[i] := byte_sign(src2.byte[i], src1.byte[i])
DEST[MAXVL-1:VL] := 0

```

### VPSIGNW dest, src1, src2 // AVX 128-bit or 256-bit Operands

```
VL=(128,256)
KL := VL/16
FOR i in 0...KL-1:
    dest.word[i] := word_sign(src2.word[i], src1.word[i])
DEST[MAXVL-1:VL] := 0

```

### VPSIGND dest, src1, src2 // AVX 128-bit or 256-bit Operands

```
VL=(128,256)
KL := VL/32
FOR i in 0...KL-1:
    dest.dword[i] := dword_sign(src2.dword[i], src1.dword[i])
DEST[MAXVL-1:VL] := 0

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
PSIGNB __m64 _mm_sign_pi8 (__m64 a, __m64 b)

```

```
(V)PSIGNB __m128i _mm_sign_epi8 (__m128i a, __m128i b)

```

```
VPSIGNB __m256i _mm256_sign_epi8 (__m256i a, __m256i b)

```

```
PSIGNW __m64 _mm_sign_pi16 (__m64 a, __m64 b)

```

```
(V)PSIGNW __m128i _mm_sign_epi16 (__m128i a, __m128i b)

```

```
VPSIGNW __m256i _mm256_sign_epi16 (__m256i a, __m256i b)

```

```
PSIGND __m64 _mm_sign_pi32 (__m64 a, __m64 b)

```

```
(V)PSIGND __m128i _mm_sign_epi32 (__m128i a, __m128i b)

```

```
VPSIGND __m256i _mm256_sign_epi32 (__m256i a, __m256i b)

```

## SIMD Floating-Point Exceptions

None.

## Other Exceptions

See Table 2-21, “Type 4 Class Exception Conditions,” additionally:

| #​​​UD | If VEX.L = 1. |
| ------ | ------------- |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
