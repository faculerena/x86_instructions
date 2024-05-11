#VRCP28PD
**Approximation to the Reciprocal of Packed Double Precision Floating**

| Opcode/Instruction                                                       | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                                                                                                                               |
| ------------------------------------------------------------------------ | ----- | ---------------------- | ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| EVEX.512.66.0F38.W1 CA /r VRCP28PD zmm1 {k1}{z}, zmm2/m512/m64bcst {sae} | A     | V/V                    | AVX512ER           | Computes the approximate reciprocals ( < 2^-28 relative error) of the packed double precision floating-point values in zmm2/m512/m64bcst and stores the results in zmm1. Under writemask. |

## Instruction Operand Encoding

| Op/En Tuple Type Operand 1 Operand 2 Operand 3 Operand 4 |     |     |     |     |     |
| -------------------------------------------------------- | --- | --- | --- | --- | --- |
| A Full ModRM:reg (w) ModRM:r/m (r) N/A N/A               |     |     |     |     |     |

### Description

Computes the reciprocal approximation of the float64 values in the source operand (the second operand) and store the results to the destination operand (the first operand). The approximate reciprocal is evaluated with less than 2^-28 of maximum relative error.

Denormal input values are treated as zeros and do not signal #​​​DE, irrespective of MXCSR.DAZ. Denormal results are flushed to zeros and do not signal #​​UE, irrespective of MXCSR.FTZ.

If any source element is NaN, the quietized NaN source value is returned for that element. If any source element is ±∞, ±0.0 is returned for that element. Also, if any source element is ±0.0, ±∞ is returned for that element.

The source operand is a ZMM register, a 512-bit memory location or a 512-bit vector broadcasted from a 64-bit memory location. The destination operand is a ZMM register, conditionally updated using writemask k1.

EVEX.vvvv is reserved and must be 1111b otherwise instructions will #​​​UD.

### A numerically exact implementation of VRCP28xx can be found at https://software.intel.com/en-us/articles/refer-

### ence-implementations-for-IA-approximation-instructions-vrcp14-vrsqrt14-vrcp28-vrsqrt28-vexp2.

### Operation

#### VRCP28PD (EVEX Encoded Versions)

```
(KL, VL) = (8, 512)
FOR j := 0 TO KL-1
    i := j * 64
    IF k1[j] OR *no writemask* THEN
            IF (EVEX.b = 1) AND (SRC *is memory*)
                THEN DEST[i+63:i] := RCP_28_DP(1.0/SRC[63:0]);
                ELSE DEST[i+63:i] := RCP_28_DP(1.0/SRC[i+63:i]);
            FI;
    ELSE
        IF *merging-masking* ; merging-masking
            THEN *DEST[i+63:i] remains unchanged*
            ELSE ; zeroing-masking
                DEST[i+63:i] := 0
        FI;
    FI;
ENDFOR;

```

| Input Value      | Result Value | Comments                                         |
| ---------------- | ------------ | ------------------------------------------------ |
| NAN              | QNAN(input)  | If (SRC = SNaN) then #​I                         |
| 0 ≤ X < 2-1022   | INF          | Positive input denormal or zero; #​Z             |
| -2-1022 < X ≤ -0 | -INF         | Negative input denormal or zero; #​Z             |
| X > 21022        | +0.0f        |                                                  |
| X < -21022       | -0.0f        |                                                  |
| X = +∞           | +0.0f        |                                                  |
| X = -∞           | -0.0f        |                                                  |
| X = 2-n          | 2n           | Exact result (unless input/output is a denormal) |
| X = -2-n         | -2n          | Exact result (unless input/output is a denormal) |

[Table 6-46](/x86/vrcp28pd#tbl-6-46). VRCP28PD Special Cases

### Intel C/C++ Compiler Intrinsic Equivalent

```
VRCP28PD __m512d _mm512_rcp28_round_pd ( __m512d a, int sae);

```

```
VRCP28PD __m512d _mm512_mask_rcp28_round_pd(__m512d a, __mmask8 m, __m512d b, int sae);

```

```
VRCP28PD __m512d _mm512_maskz_rcp28_round_pd( __mmask8 m, __m512d b, int sae);

```

### SIMD Floating-Point Exceptions

Invalid (if SNaN input), Divide-by-zero.

### Other Exceptions

See Table 2-46, “Type E2 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
