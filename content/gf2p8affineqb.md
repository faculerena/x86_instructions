#GF2P8AFFINEQB
**Galois Field Affine Transformation**

| Opcode/Instruction                                                                      | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                 |
| --------------------------------------------------------------------------------------- | ----- | ---------------------- | ------------------ | ----------------------------------------------------------- |
| 66 0F3A CE /r /ib GF2P8AFFINEQB xmm1, xmm2/m128, imm8                                   | A     | V/V                    | GFNI               | Computes affine transformation in the finite field GF(2^8). |
| VEX.128.66.0F3A.W1 CE /r /ib VGF2P8AFFINEQB xmm1, xmm2, xmm3/m128, imm8                 | B     | V/V                    | AVX GFNI           | Computes affine transformation in the finite field GF(2^8). |
| VEX.256.66.0F3A.W1 CE /r /ib VGF2P8AFFINEQB ymm1, ymm2, ymm3/m256, imm8                 | B     | V/V                    | AVX GFNI           | Computes affine transformation in the finite field GF(2^8). |
| EVEX.128.66.0F3A.W1 CE /r /ib VGF2P8AFFINEQB xmm1{k1}{z}, xmm2, xmm3/m128/m64bcst, imm8 | C     | V/V                    | AVX512VL GFNI      | Computes affine transformation in the finite field GF(2^8). |
| EVEX.256.66.0F3A.W1 CE /r /ib VGF2P8AFFINEQB ymm1{k1}{z}, ymm2, ymm3/m256/m64bcst, imm8 | C     | V/V                    | AVX512VL GFNI      | Computes affine transformation in the finite field GF(2^8). |
| EVEX.512.66.0F3A.W1 CE /r /ib VGF2P8AFFINEQB zmm1{k1}{z}, zmm2, zmm3/m512/m64bcst, imm8 | C     | V/V                    | AVX512F GFNI       | Computes affine transformation in the finite field GF(2^8). |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1        | Operand 2     | Operand 3     | Operand 4 |
| ----- | ----- | ---------------- | ------------- | ------------- | --------- |
| A     | N/A   | ModRM:reg (r, w) | ModRM:r/m (r) | imm8 (r)      | N/A       |
| B     | N/A   | ModRM:reg (w)    | VEX.vvvv (r)  | ModRM:r/m (r) | imm8 (r)  |
| C     | Full  | ModRM:reg (w)    | EVEX.vvvv (r) | ModRM:r/m (r) | imm8 (r)  |

### Description

The AFFINEB instruction computes an affine transformation in the Galois Field 28. For this instruction, an affine transformation is defined by A \* x + b where “A” is an 8 by 8 bit matrix, and “x” and “b” are 8-bit vectors. One SIMD register (operand 1) holds “x” as either 16, 32 or 64 8-bit vectors. A second SIMD (operand 2) register or memory operand contains 2, 4, or 8 “A” values, which are operated upon by the correspondingly aligned 8 “x” values in the first register. The “b” vector is constant for all calculations and contained in the immediate byte.

The EVEX encoded form of this instruction does not support memory fault suppression. The SSE encoded forms of the instruction require16B alignment on their memory operations.

### Operation

```
define parity(x):
    t := 0 // single bit
    FOR i := 0 to 7:
        t = t xor x.bit[i]
    return t
define affine_byte(tsrc2qw, src1byte, imm):
    FOR i := 0 to 7:
        * parity(x) = 1 if x has an odd number of 1s in it, and 0 otherwise.*
        retbyte.bit[i] := parity(tsrc2qw.byte[7-i] AND src1byte) XOR imm8.bit[i]
    return retbyte

```

#### VGF2P8AFFINEQB dest, src1, src2, imm8 (EVEX Encoded Version)

```
(KL, VL) = (2, 128), (4, 256), (8, 512)
FOR j := 0 TO KL-1:
    IF SRC2 is memory and EVEX.b==1:
        tsrc2 := SRC2.qword[0]
    ELSE:
        tsrc2 := SRC2.qword[j]
    FOR b := 0 to 7:
        IF k1[j*8+b] OR *no writemask*:
            DEST.qword[j].byte[b] := affine_byte(tsrc2, SRC1.qword[j].byte[b], imm8)
        ELSE IF *zeroing*:
            DEST.qword[j].byte[b] := 0
        *ELSE DEST.qword[j].byte[b] remains unchanged*
DEST[MAX_VL-1:VL] := 0

```

#### VGF2P8AFFINEQB dest, src1, src2, imm8 (128b and 256b VEX Encoded Versions)

```
(KL, VL) = (2, 128), (4, 256)
FOR j := 0 TO KL-1:
    FOR b := 0 to 7:
        DEST.qword[j].byte[b] := affine_byte(SRC2.qword[j], SRC1.qword[j].byte[b], imm8)
DEST[MAX_VL-1:VL] := 0

```

#### GF2P8AFFINEQB srcdest, src1, imm8 (128b SSE Encoded Version)

```
FOR j := 0 TO 1:
    FOR b := 0 to 7:
        SRCDEST.qword[j].byte[b] := affine_byte(SRC1.qword[j], SRCDEST.qword[j].byte[b], imm8)

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
(V)GF2P8AFFINEQB __m128i _mm_gf2p8affine_epi64_epi8(__m128i, __m128i, int);

```

```
(V)GF2P8AFFINEQB __m128i _mm_mask_gf2p8affine_epi64_epi8(__m128i, __mmask16, __m128i, __m128i, int);

```

```
(V)GF2P8AFFINEQB __m128i _mm_maskz_gf2p8affine_epi64_epi8(__mmask16, __m128i, __m128i, int);

```

```
VGF2P8AFFINEQB __m256i _mm256_gf2p8affine_epi64_epi8(__m256i, __m256i, int);

```

```
VGF2P8AFFINEQB __m256i _mm256_mask_gf2p8affine_epi64_epi8(__m256i, __mmask32, __m256i, __m256i, int);

```

```
VGF2P8AFFINEQB __m256i _mm256_maskz_gf2p8affine_epi64_epi8(__mmask32, __m256i, __m256i, int);

```

```
VGF2P8AFFINEQB __m512i _mm512_gf2p8affine_epi64_epi8(__m512i, __m512i, int);

```

```
VGF2P8AFFINEQB __m512i _mm512_mask_gf2p8affine_epi64_epi8(__m512i, __mmask64, __m512i, __m512i, int);

```

```
VGF2P8AFFINEQB __m512i _mm512_maskz_gf2p8affine_epi64_epi8(__mmask64, __m512i, __m512i, int);

```

### SIMD Floating-Point Exceptions

None.

### Other Exceptions

Legacy-encoded and VEX-encoded: See Table 2-21, “Type 4 Class Exception Conditions.”

EVEX-encoded: See Table 2-50, “Type E4NF Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
