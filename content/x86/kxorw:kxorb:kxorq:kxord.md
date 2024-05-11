# KXORW/KXORB/KXORQ/KXORD

**Bitwise Logical XOR Masks**

| Opcode/Instruction                     | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                |
| -------------------------------------- | ----- | ---------------------- | ------------------ | ---------------------------------------------------------- |
| VEX.L1.0F.W0 47 /r KXORW k1, k2, k3    | RVR   | V/V                    | AVX512F            | Bitwise XOR 16-bit masks k2 and k3 and place result in k1. |
| VEX.L1.66.0F.W0 47 /r KXORB k1, k2, k3 | RVR   | V/V                    | AVX512DQ           | Bitwise XOR 8-bit masks k2 and k3 and place result in k1.  |
| VEX.L1.0F.W1 47 /r KXORQ k1, k2, k3    | RVR   | V/V                    | AVX512BW           | Bitwise XOR 64-bit masks k2 and k3 and place result in k1. |
| VEX.L1.66.0F.W1 47 /r KXORD k1, k2, k3 | RVR   | V/V                    | AVX512BW           | Bitwise XOR 32-bit masks k2 and k3 and place result in k1. |

## Instruction Operand Encoding

| Op/En | Operand 1     | Operand 2    | Operand 3                              |
| ----- | ------------- | ------------ | -------------------------------------- |
| RVR   | ModRM:reg (w) | VEX.1vvv (r) | ModRM:r/m (r, ModRM:[7:6] must be 11b) |

## Description

Performs a bitwise XOR between the vector mask k2 and the vector mask k3, and writes the result into vector mask k1 (three-operand form).

## Operation

### KXORW

```
DEST[15:0] := SRC1[15:0] BITWISE XOR SRC2[15:0]
DEST[MAX_KL-1:16] := 0

```

### KXORB

```
DEST[7:0] := SRC1[7:0] BITWISE XOR SRC2[7:0]
DEST[MAX_KL-1:8] := 0

```

### KXORQ

```
DEST[63:0] := SRC1[63:0] BITWISE XOR SRC2[63:0]
DEST[MAX_KL-1:64] := 0

```

### KXORD

```
DEST[31:0] := SRC1[31:0] BITWISE XOR SRC2[31:0]
DEST[MAX_KL-1:32] := 0

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
KXORW __mmask16 _mm512_kxor(__mmask16 a, __mmask16 b);

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
