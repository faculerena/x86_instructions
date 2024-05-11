# WRPKRU

**Write Data to User Page Key Register**

| Opcode/Instruction | Op/En | 64/32bit Mode Support | CPUID Feature Flag | Description           |
| ------------------ | ----- | --------------------- | ------------------ | --------------------- |
| NP 0F 01 EF WRPKRU | ZO    | V/V                   | OSPKE              | Writes EAX into PKRU. |

## Instruction Operand Encoding

| Op/En | Operand 1 | Operand 2 | Operand 3 | Operand 4 |
| ----- | --------- | --------- | --------- | --------- |
| ZO    | N/A       | N/A       | N/A       | N/A       |

## Description

Writes the value of EAX into PKRU. ECX and EDX must be 0 when WRPKRU is executed; otherwise, a general-protection exception (#​​​​GP) occurs.

WRPKRU can be executed only if CR4.PKE = 1; otherwise, an invalid-opcode exception (#​​​UD) occurs. Software can discover the value of CR4.PKE by examining CPUID.(EAX=07H,ECX=0H):ECX.OSPKE [bit 4].

On processors that support the Intel 64 Architecture, the high-order 32-bits of RCX, RDX, and RAX are ignored.

WRPKRU will never execute speculatively. Memory accesses affected by PKRU register will not execute (even speculatively) until all prior executions of WRPKRU have completed execution and updated the PKRU register.

## Operation

```
IF (ECX = 0 AND EDX = 0)
    THEN PKRU := EAX;
    ELSE #​​​​GP(0);
FI;

```

## Flags Affected

None.

## C/C++ Compiler Intrinsic Equivalent

```
WRPKRU void _wrpkru(uint32_t);

```

## Protected Mode Exceptions

| \#​​​​GP(0)     | If ECX ≠ 0.                 |
| --------------- | --------------------------- |
| If EDX ≠ 0.     |
| #​​​UD          | If the LOCK prefix is used. |
| If CR4.PKE = 0. |

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
