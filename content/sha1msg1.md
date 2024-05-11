# SHA1MSG1

**Perform an Intermediate Calculation for the Next Four SHA**

| Opcode/Instruction                      | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                                                                                                   |
| --------------------------------------- | ----- | ---------------------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| NP 0F 38 C9 /r SHA1MSG1 xmm1, xmm2/m128 | RM    | V/V                    | SHA                | Performs an intermediate calculation for the next four SHA1 message dwords using previous message dwords from xmm1 and xmm2/m128, storing the result in xmm1. |

## Instruction Operand Encoding

| Op/En | Operand 1        | Operand 2     | Operand 3 |
| ----- | ---------------- | ------------- | --------- |
| RM    | ModRM:reg (r, w) | ModRM:r/m (r) | N/A       |

## Description

The SHA1MSG1 instruction is one of two SHA1 message scheduling instructions. The instruction performs an intermediate calculation for the next four SHA1 message dwords.

## Operation

### SHA1MSG1

```
W0 := SRC1[127:96] ;
W1 := SRC1[95:64] ;
W2 := SRC1[63: 32] ;
W3 := SRC1[31: 0] ;
W4 := SRC2[127:96] ;
W5 := SRC2[95:64] ;
DEST[127:96] := W2 XOR W0;
DEST[95:64] := W3 XOR W1;
DEST[63:32] := W4 XOR W2;
DEST[31:0] := W5 XOR W3;

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
SHA1MSG1 __m128i _mm_sha1msg1_epu32(__m128i, __m128i);

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
