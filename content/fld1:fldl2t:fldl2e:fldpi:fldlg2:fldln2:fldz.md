# FLD1/FLDL2T/FLDL2E/FLDPI/FLDLG2/FLDLN2/FLDZ

**Load Constant**

| Opcode\* | Instruction | 64-Bit Mode | Compat/Leg Mode | Description                              |
| -------- | ----------- | ----------- | --------------- | ---------------------------------------- |
| D9 E8    | FLD1        | Valid       | Valid           | Push +1.0 onto the FPU register stack.   |
| D9 E9    | FLDL2T      | Valid       | Valid           | Push log210 onto the FPU register stack. |
| D9 EA    | FLDL2E      | Valid       | Valid           | Push log2e onto the FPU register stack.  |
| D9 EB    | FLDPI       | Valid       | Valid           | Push π onto the FPU register stack.      |
| D9 EC    | FLDLG2      | Valid       | Valid           | Push log102 onto the FPU register stack. |
| D9 ED    | FLDLN2      | Valid       | Valid           | Push loge2 onto the FPU register stack.  |
| D9 EE    | FLDZ        | Valid       | Valid           | Push +0.0 onto the FPU register stack.   |

> \* SeeIA-32ArchitectureCompatibilitysectionbelow.

## Description

Push one of seven commonly used constants (in double extended-precision floating-point format) onto the FPU register stack. The constants that can be loaded with these instructions include +1.0, +0.0, log210, log2e, π, log102, and loge2. For each constant, an internal 66-bit constant is rounded (as specified by the RC field in the FPU control word) to double extended-precision floating-point format. The inexact-result exception (#​P) is not generated as a result of the rounding, nor is the C1 flag set in the x87 FPU status word if the value is rounded up.

See the section titled “Approximation of Pi” in Chapter 8 of the Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 1, for a description of the π constant.

This instruction’s operation is the same in non-64-bit modes and 64-bit mode.

## IA-32 Architecture Compatibility

When the RC field is set to round-to-nearest, the FPU produces the same constants that is produced by the Intel 8087 and Intel 287 math coprocessors.

## Operation

```
TOP := TOP − 1;
ST(0) := CONSTANT;

```

## FPU Flags Affected

| C1         | Set to 1 if stack overflow occurred; otherwise, set to 0. |
| ---------- | --------------------------------------------------------- |
| C0, C2, C3 | Undefined.                                                |

## Floating-Point Exceptions

| \#​IS | Stack overflow occurred. |
| ----- | ------------------------ |

## Protected Mode Exceptions

| \#​NM  | CR0.EM[bit 2] or CR0.TS[bit 3] = 1.      |
| ------ | ---------------------------------------- |
| \#​​MF | If there is a pending x87 FPU exception. |
| #​​​UD | If the LOCK prefix is used.              |

## Real-Address Mode Exceptions

Same exceptions as in protected mode.

## Virtual-8086 Mode Exceptions

Same exceptions as in protected mode.

## Virtual-8086 Mode Exceptions

Same exceptions as in protected mode.

## Compatibility Mode Exceptions

Same exceptions as in protected mode.

## 64-Bit Mode Exceptions

Same exceptions as in protected mode.

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
