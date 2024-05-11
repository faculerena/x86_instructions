# FABS

**Absolute Value**

| Opcode |     | Mode | Leg Mode | Description                         |
| ------ | --- | ---- | -------- | ----------------------------------- |
| D9 E1  |     |      |          | Replace ST with its absolute value. |

## Description

Clears the sign bit of ST(0) to create the absolute value of the operand. The following table shows the results obtained when creating the absolute value of various classes of numbers.

| ST(0) SRC | ST(0) DEST |
| --------- | ---------- |
| −∞        | +∞         |
| −F        | +F         |
| −0        | +0         |
| +0        | +0         |
| +F        | +F         |
| +∞        | +∞         |
| NaN       | NaN        |

[Table 3-17](/x86/fabs#tbl-3-17). Results Obtained from FABS

> F Meansfinitefloating-pointvalue.

This instruction’s operation is the same in non-64-bit modes and 64-bit mode.

## Operation

```
ST(0) := |ST(0)|;

```

## FPU Flags Affected

| C1         | Set to 0.  |
| ---------- | ---------- |
| C0, C2, C3 | Undefined. |

## Floating-Point Exceptions

| \#​IS | Stack underflow occurred. |
| ----- | ------------------------- |

## Protected Mode Exceptions

| \#​NM  | CR0.EM[bit 2] or CR0.TS[bit 3] = 1. |
| ------ | ----------------------------------- |
| #​​​UD | If the LOCK prefix is used.         |

## Real-Address Mode Exceptions

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
