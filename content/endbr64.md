# ENDBR64

**Terminate an Indirect Branch in **

| Opcode/Instruction  | Op / En | 64/32 bit Mode Support | CPUID Feature Flag | Description                               |
| ------------------- | ------- | ---------------------- | ------------------ | ----------------------------------------- |
| F3 0F 1E FA ENDBR64 | ZO      | V/V                    | CET_IBT            | Terminate indirect branch in 64-bit mode. |

## Instruction Operand Encoding

| Op/En | Tuple Type | Operand 1 | Operand 2 | Operand 3 | Operand 4 |
| ----- | ---------- | --------- | --------- | --------- | --------- |
| ZO    | N/A        | N/A       | N/A       | N/A       | N/A       |

## Description

Terminate an indirect branch in 64 bit mode.

## Operation

```
IF EndbranchEnabled(CPL) & IA32_EFER.LMA = 1 & CS.L = 1
    IF CPL = 3
        THEN
            IA32_U_CET.TRACKER = IDLE
            IA32_U_CET.SUPPRESS = 0
        ELSE
            IA32_S_CET.TRACKER = IDLE
            IA32_S_CET.SUPPRESS = 0
    FI;
FI;

```

## Flags Affected

None.

## Exceptions

None.

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
