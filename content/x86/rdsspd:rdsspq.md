# RDSSPD/RDSSPQ

**Read Shadow Stack Pointer**

| Opcode/Instruction                    | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                            |
| ------------------------------------- | ----- | ---------------------- | ------------------ | ------------------------------------------------------ |
| F3 0F 1E /1 (mod=11) RDSSPD r32       | R     | V/V                    | CET_SS             | Copy low 32 bits of shadow stack pointer (SSP) to r32. |
| F3 REX.W 0F 1E /1 (mod=11) RDSSPQ r64 | R     | V/N.E.                 | CET_SS             | Copies shadow stack pointer (SSP) to r64.              |

## Instruction Operand Encoding

| Op/En | Operand 1     | Operand 2 | Operand 3 | Operand 4 |
| ----- | ------------- | --------- | --------- | --------- |
| R     | ModRM:r/m (w) | N/A       | N/A       | N/A       |

## Description

Copies the current shadow stack pointer (SSP) register to the register destination. This opcode is a NOP when CET shadow stacks are not enabled and on processors that do not support CET.

## Operation

```
IF CPL = 3
    IF CR4.CET & IA32_U_CET.SH_STK_EN
        IF (operand size is 64 bit)
            THEN
                Dest := SSP;
            ELSE
                Dest := SSP[31:0];
        FI;
    FI;
ELSE
    IF CR4.CET & IA32_S_CET.SH_STK_EN
        IF (operand size is 64 bit)
            THEN
                Dest := SSP;
            ELSE
                Dest := SSP[31:0];
        FI;
    FI;
FI;

```

## Flags Affected

None.

## C/C++ Compiler Intrinsic Equivalent

```
RDSSPD__int32 _rdsspd_i32(void);

```

```
RDSSPQ__int64 _rdsspq_i64(void);

```

## Protected Mode Exceptions

None.

## Real-Address Mode Exceptions

None.

## Virtual-8086 Mode Exceptions

None.

## Compatibility Mode Exceptions

None.

## 64-Bit Mode Exceptions

None.

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
