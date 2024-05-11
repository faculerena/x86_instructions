# LAHF

**Load Status Flags Into AH Register**

| Opcode |     | En  | Mode | Leg Mode | Description                               |
| ------ | --- | --- | ---- | -------- | ----------------------------------------- |
| 9F     |     |     |      |          | Load: AH := EFLAGS(SF:ZF:0:AF:0:PF:1:CF). |

1. Valid in specific steppings; see Description section.

## Instruction Operand Encoding

| Op/En | Operand 1 | Operand 2 | Operand 3 | Operand 4 |
| ----- | --------- | --------- | --------- | --------- |
| ZO    | N/A       | N/A       | N/A       | N/A       |

## Description

This instruction executes as described above in compatibility mode and legacy mode. It is valid in 64-bit mode only if CPUID.80000001H:ECX.LAHF-SAHF[bit 0] = 1.

## Operation

```
IF 64-Bit Mode
    THEN
        IF CPUID.80000001H:ECX.LAHF-SAHF[bit 0] = 1;
            THEN AH := RFLAGS(SF:ZF:0:AF:0:PF:1:CF);
            ELSE #​​​UD;
        FI;
    ELSE
        AH := EFLAGS(SF:ZF:0:AF:0:PF:1:CF);
FI;

```

## Flags Affected

None. The state of the flags in the EFLAGS register is not affected.

## Protected Mode Exceptions

| #​​​UD | If the LOCK prefix is used. |
| ------ | --------------------------- |

## Real-Address Mode Exceptions

Same exceptions as in protected mode.

## Virtual-8086 Mode Exceptions

Same exceptions as in protected mode.

## Compatibility Mode Exceptions

Same exceptions as in protected mode.

## 64-Bit Mode Exceptions

| #​​​UD                      | If CPUID.80000001H:ECX.LAHF-SAHF[bit 0] = 0. |
| --------------------------- | -------------------------------------------- |
| If the LOCK prefix is used. |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
