#HRESET
**History Reset**

| Opcode/Instruction                    | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                              |
| ------------------------------------- | ----- | ---------------------- | ------------------ | ------------------------------------------------------------------------ |
| F3 0F 3A F0 C0 /ib HRESET imm8, <EAX> | A     | V/V                    | HRESET             | Processor history reset request. Controlled by the EAX implicit operand. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1     | Operand 2 | Operand 3 | Operand 4 |
| ----- | ----- | ------------- | --------- | --------- | --------- |
| A     | N/A   | ModRM:r/m (r) | N/A       | N/A       | N/A       |

### Description

Requests the processor to selectively reset selected components of hardware history maintained by the current logical processor. HRESET operation is controlled by the implicit EAX operand. The value of the explicit imm8 operand is ignored. This instruction can only be executed at privilege level 0.

The HRESET instruction can be used to request reset of multiple components of hardware history. Prior to the execution of HRESET, the system software must take the following steps:

1. Enumerate the HRESET capabilities via CPUID.20H.0H:EBX, which indicates what components of hardware history can be reset.

2. Only the bits enumerated by CPUID.20H.0H:EBX can be set in the IA32_HRESET_ENABLE MSR.

HRESET causes a general-protection exception (#​​​​GP) if EAX sets any bits that are not set in the IA32_HRESET_EN-ABLE MSR.

Any attempt to execute the HRESET instruction inside a transactional region will result in a transaction abort.

### Operation

```
IF EAX = 0
    THEN NOP
    ELSE
        FOREACH i such that EAX[i] = 1
            Reset prediction history for feature i
FI

```

### Flags Affected

None.

## Protected Mode Exceptions

| \#​​​​GP(0) | If CPL > 0 or (EAX AND NOT IA32_HRESET_ENABLE) ≠0. |
| ----------- | -------------------------------------------------- |
| #​​​UD      | If CPUID.07H.01H:EAX.HRESET[bit 22] = 0.           |

## Real-Address Mode Exceptions

Same exceptions as in protected mode.

## Virtual-8086 Mode Exceptions

| \#​​​​GP(0) | HRESET instruction is not recognized in virtual-8086 mode. |
| ----------- | ---------------------------------------------------------- |

## Compatibility Mode Exceptions

Same exceptions as in protected mode.

## 64-Bit Mode Exceptions

Same exceptions as in protected mode.

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
