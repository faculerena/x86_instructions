#KADDW/KADDB/KADDQ/KADDD
**ADD Two Masks**

| Opcode/Instruction                     | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                            |
| -------------------------------------- | ----- | ---------------------- | ------------------ | ------------------------------------------------------ |
| VEX.L1.0F.W0 4A /r KADDW k1, k2, k3    | RVR   | V/V                    | AVX512DQ           | Add 16 bits masks in k2 and k3 and place result in k1. |
| VEX.L1.66.0F.W0 4A /r KADDB k1, k2, k3 | RVR   | V/V                    | AVX512DQ           | Add 8 bits masks in k2 and k3 and place result in k1.  |
| VEX.L1.0F.W1 4A /r KADDQ k1, k2, k3    | RVR   | V/V                    | AVX512BW           | Add 64 bits masks in k2 and k3 and place result in k1. |
| VEX.L1.66.0F.W1 4A /r KADDD k1, k2, k3 | RVR   | V/V                    | AVX512BW           | Add 32 bits masks in k2 and k3 and place result in k1. |

## Instruction Operand Encoding

| Op/En | Operand 1     | Operand 2    | Operand 3                              |
| ----- | ------------- | ------------ | -------------------------------------- |
| RVR   | ModRM:reg (w) | VEX.1vvv (r) | ModRM:r/m (r, ModRM:[7:6] must be 11b) |

## Description

Adds the vector mask k2 and the vector mask k3, and writes the result into vector mask k1.

## Operation

### KADDW

```
DEST[15:0] := SRC1[15:0] + SRC2[15:0]
DEST[MAX_KL-1:16] := 0

```

### KADDB

```
DEST[7:0] := SRC1[7:0] + SRC2[7:0]
DEST[MAX_KL-1:8] := 0

```

### KADDQ

```
DEST[63:0] := SRC1[63:0] + SRC2[63:0]
DEST[MAX_KL-1:64] := 0

```

### KADDD

```
DEST[31:0] := SRC1[31:0] + SRC2[31:0]
DEST[MAX_KL-1:32] := 0

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
KADDW __mmask16 _kadd_mask16 (__mmask16 a, __mmask16 b);

```

```
KADDB __mmask8 _kadd_mask8 (__mmask8 a, __mmask8 b);

```

```
KADDQ __mmask64 _kadd_mask64 (__mmask64 a, __mmask64 b);

```

```
KADDD __mmask32 _kadd_mask32 (__mmask32 a, __mmask32 b);

```

## Flags Affected

None.

## SIMD Floating-Point Exceptions

None.

## Other Exceptions

See Table 2-63, “TYPE K20 Exception Definition (VEX-Encoded OpMask Instructions w/o Memory Arg).”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
