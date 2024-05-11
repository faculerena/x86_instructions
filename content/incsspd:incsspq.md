# INCSSPD/INCSSPQ

**Increment Shadow Stack Pointer**

| Opcode/Instruction             | Op / En | 64/32 bit Mode Support | CPUID Feature Flag | Description                     |
| ------------------------------ | ------- | ---------------------- | ------------------ | ------------------------------- |
| F3 0F AE /05 INCSSPD r32       | R       | V/V                    | CET_SS             | Increment SSP by 4 \* r32[7:0]. |
| F3 REX.W 0F AE /05 INCSSPQ r64 | R       | V/N.E.                 | CET_SS             | Increment SSP by 8 \* r64[7:0]. |

## Instruction Operand Encoding

| Op/En | Tuple Type | Operand 1     | Operand 2 | Operand 3 | Operand 4 |
| ----- | ---------- | ------------- | --------- | --------- | --------- |
| R     | N/A        | ModRM:r/m (r) | N/A       | N/A       | N/A       |

## Description

This instruction can be used to increment the current shadow stack pointer by the operand size of the instruction times the unsigned 8-bit value specified by bits 7:0 in the source operand. The instruction performs a pop and discard of the first and last element on the shadow stack in the range specified by the unsigned 8-bit value in bits 7:0 of the source operand.

## Operation

```
IF CPL = 3
    IF (CR4.CET & IA32_U_CET.SH_STK_EN) = 0
        THEN #​​​UD; FI;
ELSE
    IF (CR4.CET & IA32_S_CET.SH_STK_EN) = 0
        THEN #​​​UD; FI;
FI;
IF (operand size is 64-bit)
    THEN
        Range := R64[7:0];
        shadow_stack_load 8 bytes from SSP;
        IF Range > 0
            THEN shadow_stack_load 8 bytes from SSP + 8 * (Range - 1);
        FI;
        SSP := SSP + Range * 8;
    ELSE
        Range := R32[7:0];
        shadow_stack_load 4 bytes from SSP;
        IF Range > 0
            THEN shadow_stack_load 4 bytes from SSP + 4 * (Range - 1);
        FI;
        SSP := SSP + Range * 4;
FI;

```

## Flags Affected

None.

## Intel C/C++ Compiler Intrinsic Equivalent

```
INCSSPD void _incsspd(int);

```

```
INCSSPQ void _incsspq(int);

```

## Protected Mode Exceptions

| #​​​UD                                   | If the LOCK prefix is used. |
| ---------------------------------------- | --------------------------- |
| If CR4.CET = 0.                          |
| IF CPL = 3 and IA32_U_CET.SH_STK_EN = 0. |
| IF CPL < 3 and IA32_S_CET.SH_STK_EN = 0. |
| \#​PF(fault-code)                        | If a page fault occurs.     |

## Real-Address Mode Exceptions

| #​​​UD | The INCSSP instruction is not recognized in real-address mode. |
| ------ | -------------------------------------------------------------- |

## Virtual-8086 Mode Exceptions

| #​​​UD | The INCSSP instruction is not recognized in virtual-8086 mode. |
| ------ | -------------------------------------------------------------- |

## Compatibility Mode Exceptions

Same exceptions as in protected mode.

## 64-Bit Mode Exceptions

Same exceptions as in protected mode.

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
