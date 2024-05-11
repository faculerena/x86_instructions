#SHA1NEXTE
**Calculate SHA**

| Opcode/Instruction                       | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                                                                                                                                                                                                            |
| ---------------------------------------- | ----- | ---------------------- | ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| NP 0F 38 C8 /r SHA1NEXTE xmm1, xmm2/m128 | RM    | V/V                    | SHA                | Calculates SHA1 state variable E after four rounds of operation from the current SHA1 state variable A in xmm1. The calculated value of the SHA1 state variable E is added to the scheduled dwords in xmm2/m128, and stored with some of the scheduled dwords in xmm1. |

## Instruction Operand Encoding

| Op/En | Operand 1        | Operand 2     | Operand 3 |
| ----- | ---------------- | ------------- | --------- |
| RM    | ModRM:reg (r, w) | ModRM:r/m (r) | N/A       |

## Description

The SHA1NEXTE calculates the SHA1 state variable E after four rounds of operation from the current SHA1 state variable A in the destination operand. The calculated value of the SHA1 state variable E is added to the source operand, which contains the scheduled dwords.

## Operation

### SHA1NEXTE

```
TMP := (SRC1[127:96] ROL 30);
DEST[127:96] := SRC2[127:96] + TMP;
DEST[95:64] := SRC2[95:64];
DEST[63:32] := SRC2[63:32];
DEST[31:0] := SRC2[31:0];

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
SHA1NEXTE __m128i _mm_sha1nexte_epu32(__m128i, __m128i);

```

## Flags Affected

None.

## SIMD Floating-Point Exceptions

None.

## Other Exceptions

See Table 2-21, “Type 4 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
