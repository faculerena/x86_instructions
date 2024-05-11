#GF2P8AFFINEINVQB
**Galois Field Affine Transformation Inverse**

| Opcode/Instruction                                                                         | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                         |
| ------------------------------------------------------------------------------------------ | ----- | ---------------------- | ------------------ | ------------------------------------------------------------------- |
| 66 0F3A CF /r /ib GF2P8AFFINEINVQB xmm1, xmm2/m128, imm8                                   | A     | V/V                    | GFNI               | Computes inverse affine transformation in the finite field GF(2^8). |
| VEX.128.66.0F3A.W1 CF /r /ib VGF2P8AFFINEINVQB xmm1, xmm2, xmm3/m128, imm8                 | B     | V/V                    | AVX GFNI           | Computes inverse affine transformation in the finite field GF(2^8). |
| VEX.256.66.0F3A.W1 CF /r /ib VGF2P8AFFINEINVQB ymm1, ymm2, ymm3/m256, imm8                 | B     | V/V                    | AVX GFNI           | Computes inverse affine transformation in the finite field GF(2^8). |
| EVEX.128.66.0F3A.W1 CF /r /ib VGF2P8AFFINEINVQB xmm1{k1}{z}, xmm2, xmm3/m128/m64bcst, imm8 | C     | V/V                    | AVX512VL GFNI      | Computes inverse affine transformation in the finite field GF(2^8). |
| EVEX.256.66.0F3A.W1 CF /r /ib VGF2P8AFFINEINVQB ymm1{k1}{z}, ymm2, ymm3/m256/m64bcst, imm8 | C     | V/V                    | AVX512VL GFNI      | Computes inverse affine transformation in the finite field GF(2^8). |
| EVEX.512.66.0F3A.W1 CF /r /ib VGF2P8AFFINEINVQB zmm1{k1}{z}, zmm2, zmm3/m512/m64bcst, imm8 | C     | V/V                    | AVX512F GFNI       | Computes inverse affine transformation in the finite field GF(2^8). |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1        | Operand 2     | Operand 3     | Operand 4 |
| ----- | ----- | ---------------- | ------------- | ------------- | --------- |
| A     | N/A   | ModRM:reg (r, w) | ModRM:r/m (r) | imm8 (r)      | N/A       |
| B     | N/A   | ModRM:reg (w)    | VEX.vvvv (r)  | ModRM:r/m (r) | imm8 (r)  |
| C     | Full  | ModRM:reg (w)    | EVEX.vvvv (r) | ModRM:r/m (r) | imm8 (r)  |

### Description

The AFFINEINVB instruction computes an affine transformation in the Galois Field 28. For this instruction, an affine transformation is defined by A \* inv(x) + b where “A” is an 8 by 8 bit matrix, and “x” and “b” are 8-bit vectors. The inverse of the bytes in x is defined with respect to the reduction polynomial x8 + x4 + x3 + x + 1.

One SIMD register (operand 1) holds “x” as either 16, 32 or 64 8-bit vectors. A second SIMD (operand 2) register or memory operand contains 2, 4, or 8 “A” values, which are operated upon by the correspondingly aligned 8 “x” values in the first register. The “b” vector is constant for all calculations and contained in the immediate byte.

The EVEX encoded form of this instruction does not support memory fault suppression. The SSE encoded forms of the instruction require 16B alignment on their memory operations.

The inverse of each byte is given by the following table. The upper nibble is on the vertical axis and the lower nibble is on the horizontal axis. For example, the inverse of 0x95 is 0x8A.

| -   | 0   | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   | 9   | A   | B   | C   | D   | E   | F   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0   | 0   | 1   | 8D  | F6  | CB  | 52  | 7B  | D1  | E8  | 4F  | 29  | C0  | B0  | E1  | E5  | C7  |
| 1   | 74  | B4  | AA  | 4B  | 99  | 2B  | 60  | 5F  | 58  | 3F  | FD  | CC  | FF  | 40  | EE  | B2  |
| 2   | 3A  | 6E  | 5A  | F1  | 55  | 4D  | A8  | C9  | C1  | A   | 98  | 15  | 30  | 44  | A2  | C2  |
| 3   | 2C  | 45  | 92  | 6C  | F3  | 39  | 66  | 42  | F2  | 35  | 20  | 6F  | 77  | BB  | 59  | 19  |
| 4   | 1D  | FE  | 37  | 67  | 2D  | 31  | F5  | 69  | A7  | 64  | AB  | 13  | 54  | 25  | E9  | 9   |
| 5   | ED  | 5C  | 5   | CA  | 4C  | 24  | 87  | BF  | 18  | 3E  | 22  | F0  | 51  | EC  | 61  | 17  |
| 6   | 16  | 5E  | AF  | D3  | 49  | A6  | 36  | 43  | F4  | 47  | 91  | DF  | 33  | 93  | 21  | 3B  |
| 7   | 79  | B7  | 97  | 85  | 10  | B5  | BA  | 3C  | B6  | 70  | D0  | 6   | A1  | FA  | 81  | 82  |
| 8   | 83  | 7E  | 7F  | 80  | 96  | 73  | BE  | 56  | 9B  | 9E  | 95  | D9  | F7  | 2   | B9  | A4  |
| 9   | DE  | 6A  | 32  | 6D  | D8  | 8A  | 84  | 72  | 2A  | 14  | 9F  | 88  | F9  | DC  | 89  | 9A  |
| A   | FB  | 7C  | 2E  | C3  | 8F  | B8  | 65  | 48  | 26  | C8  | 12  | 4A  | CE  | E7  | D2  | 62  |
| B   | C   | E0  | 1F  | EF  | 11  | 75  | 78  | 71  | A5  | 8E  | 76  | 3D  | BD  | BC  | 86  | 57  |
| C   | B   | 28  | 2F  | A3  | DA  | D4  | E4  | F   | A9  | 27  | 53  | 4   | 1B  | FC  | AC  | E6  |
| D   | 7A  | 7   | AE  | 63  | C5  | DB  | E2  | EA  | 94  | 8B  | C4  | D5  | 9D  | F8  | 90  | 6B  |
| E   | B1  | D   | D6  | EB  | C6  | E   | CF  | AD  | 8   | 4E  | D7  | E3  | 5D  | 50  | 1E  | B3  |
| F   | 5B  | 23  | 38  | 34  | 68  | 46  | 3   | 8C  | DD  | 9C  | 7D  | A0  | CD  | 1A  | 41  | 1C  |

[Table 3-50](/x86/gf2p8affineinvqb#tbl-3-50). Inverse Byte Listings

### Operation

```
define affine_inverse_byte(tsrc2qw, src1byte, imm):
    FOR i := 0 to 7:
        * parity(x) = 1 if x has an odd number of 1s in it, and 0 otherwise.*
        * inverse(x) is defined in the table above *
        retbyte.bit[i] := parity(tsrc2qw.byte[7-i] AND inverse(src1byte)) XOR imm8.bit[i]
    return retbyte

```

#### VGF2P8AFFINEINVQB dest, src1, src2, imm8 (EVEX Encoded Version)

```
(KL, VL) = (2, 128), (4, 256), (8, 512)
FOR j := 0 TO KL-1:
    IF SRC2 is memory and EVEX.b==1:
        tsrc2 := SRC2.qword[0]
    ELSE:
        tsrc2 := SRC2.qword[j]
FOR b := 0 to 7:
    IF k1[j*8+b] OR *no writemask*:
        FOR i := 0 to 7:
            DEST.qword[j].byte[b] := affine_inverse_byte(tsrc2, SRC1.qword[j].byte[b], imm8)
    ELSE IF *zeroing*:
        DEST.qword[j].byte[b] := 0
    *ELSE DEST.qword[j].byte[b] remains unchanged*
DEST[MAX_VL-1:VL] := 0

```

#### VGF2P8AFFINEINVQB dest, src1, src2, imm8 (128b and 256b VEX Encoded Versions)

```
(KL, VL) = (2, 128), (4, 256)
FOR j := 0 TO KL-1:
    FOR b := 0 to 7:
        DEST.qword[j].byte[b] := affine_inverse_byte(SRC2.qword[j], SRC1.qword[j].byte[b], imm8)
DEST[MAX_VL-1:VL] := 0

```

#### GF2P8AFFINEINVQB srcdest, src1, imm8 (128b SSE Encoded Version)

```
FOR j := 0 TO 1:
    FOR b := 0 to 7:
        SRCDEST.qword[j].byte[b] := affine_inverse_byte(SRC1.qword[j], SRCDEST.qword[j].byte[b], imm8)

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
(V)GF2P8AFFINEINVQB __m128i _mm_gf2p8affineinv_epi64_epi8(__m128i, __m128i, int);

```

```
(V)GF2P8AFFINEINVQB __m128i _mm_mask_gf2p8affineinv_epi64_epi8(__m128i, __mmask16, __m128i, __m128i, int);

```

```
(V)GF2P8AFFINEINVQB __m128i _mm_maskz_gf2p8affineinv_epi64_epi8(__mmask16, __m128i, __m128i, int);

```

```
VGF2P8AFFINEINVQB __m256i _mm256_gf2p8affineinv_epi64_epi8(__m256i, __m256i, int);

```

```
VGF2P8AFFINEINVQB __m256i _mm256_mask_gf2p8affineinv_epi64_epi8(__m256i, __mmask32, __m256i, __m256i, int);

```

```
VGF2P8AFFINEINVQB __m256i _mm256_maskz_gf2p8affineinv_epi64_epi8(__mmask32, __m256i, __m256i, int);

```

```
VGF2P8AFFINEINVQB __m512i _mm512_gf2p8affineinv_epi64_epi8(__m512i, __m512i, int);

```

```
VGF2P8AFFINEINVQB __m512i _mm512_mask_gf2p8affineinv_epi64_epi8(__m512i, __mmask64, __m512i, __m512i, int);

```

```
VGF2P8AFFINEINVQB __m512i _mm512_maskz_gf2p8affineinv_epi64_epi8(__mmask64, __m512i, __m512i, int);

```

### SIMD Floating-Point Exceptions

None.

### Other Exceptions

Legacy-encoded and VEX-encoded: See Table 2-21, “Type 4 Class Exception Conditions.”

EVEX-encoded: See Table 2-50, “Type E4NF Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
