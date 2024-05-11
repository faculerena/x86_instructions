# TESTUI**Determine User Interrupt Flag**

| Opcode/Instruction | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                     |
| ------------------ | ----- | ---------------------- | ------------------ | ----------------------------------------------- |
| F3 0F 01 ED TESTUI | ZO    | V/I                    | UINTR              | Copies the current value of UIF into EFLAGS.CF. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1 | Operand 2 | Operand 3 | Operand 4 |
| ----- | ----- | --------- | --------- | --------- | --------- |
| ZO    | N/A   | N/A       | N/A       | N/A       | N/A       |

TESTUI copies the current value of the user interrupt flag (UIF) into EFLAGS.CF. This instruction can be executed regardless of CPL.

TESTUI may be executed normally inside a transactional region.

## Operation

```
CF := UIF;
ZF := AF := OF := PF := SF := 0;

```

## Flags Affected

The ZF, OF, AF, PF, SF flags are cleared and the CF flags to the value of the user interrupt flag.

## Protected Mode Exceptions

| #​​​UD | The TESTUI instruction is not recognized in protected mode. |
| ------ | ----------------------------------------------------------- |

## Real-Address Mode Exceptions

| #​​​UD | The TESTUI instruction is not recognized in real-address mode. |
| ------ | -------------------------------------------------------------- |

## Virtual-8086 Mode Exceptions

| #​​​UD | The TESTUI instruction is not recognized in virtual-8086 mode. |
| ------ | -------------------------------------------------------------- |

## Compatibility Mode Exceptions

| #​​​UD | The TESTUI instruction is not recognized in compatibility mode. |
| ------ | --------------------------------------------------------------- |

## 64-Bit Mode Exceptions

| #​​​UD                                | If the LOCK prefix is used. |
| ------------------------------------- | --------------------------- |
| If executed inside an enclave.        |
| If CR4.UINTR = 0.                     |
| If CPUID.07H.0H:EDX.UINTR[bit 5] = 0. |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
