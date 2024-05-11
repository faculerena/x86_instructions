# STMXCSR**Store MXCSR Register State**

| Opcode\*/Instruction               | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                |
| ---------------------------------- | ----- | ---------------------- | ------------------ | ------------------------------------------ |
| NP 0F AE /3 STMXCSR _m32_          | M     | V/V                    | SSE                | Store contents of MXCSR register to _m32_. |
| VEX.LZ.0F.WIG AE /3 VSTMXCSR _m32_ | M     | V/V                    | AVX                | Store contents of MXCSR register to _m32_. |

## Instruction Operand Encoding

| Op/En | Operand 1     | Operand 2 | Operand 3 | Operand 4 |
| ----- | ------------- | --------- | --------- | --------- |
| M     | ModRM:r/m (w) | N/A       | N/A       | N/A       |

## Description

Stores the contents of the MXCSR control and status register to the destination operand. The destination operand is a 32-bit memory location. The reserved bits in the MXCSR register are stored as 0s.

This instruction’s operation is the same in non-64-bit modes and 64-bit mode.

VEX.L must be 0, otherwise instructions will #​​​UD.

Note: In VEX-encoded versions, VEX.vvvv is reserved and must be 1111b, otherwise instructions will #​​​UD.

## Operation

```
m32 := MXCSR;

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
_mm_getcsr(void)

```

## SIMD Floating-Point Exceptions

None.

## Other Exceptions

See Table 2-22, “Type 5 Class Exception Conditions,” additionally:

| #​​​UD               | If VEX.L= 1, |
| -------------------- | ------------ |
| If VEX.vvvv ≠ 1111B. |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
