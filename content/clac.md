# CLAC

**Clear AC Flag in EFLAGS Register**

| Opcode/Instruction | Op / En | 64/32 bit Mode Support | CPUID Feature Flag | Description                               |
| ------------------ | ------- | ---------------------- | ------------------ | ----------------------------------------- |
| NP 0F 01 CA CLAC   | ZO      | V/V                    | SMAP               | Clear the AC flag in the EFLAGS register. |

## Instruction Operand Encoding

| Op/En | Operand 1 | Operand 2 | Operand 3 | Operand 4 |
| ----- | --------- | --------- | --------- | --------- |
| ZO    | N/A       | N/A       | N/A       | N/A       |

## Description

Clears the AC flag bit in EFLAGS register. This disables any alignment checking of user-mode data accesses. Ifthe SMAP bit is set in the CR4 register, this disallows explicit supervisor-mode data accesses to user-mode pages.

This instruction's operation is the same in non-64-bit modes and 64-bit mode. Attempts to execute CLAC when CPL > 0 cause #​​​UD.

## Operation

```
EFLAGS.AC := 0;

```

## Flags Affected

AC cleared. Other flags are unaffected.

## Protected Mode Exceptions

| #​​​UD                                           | If the LOCK prefix is used. |
| ------------------------------------------------ | --------------------------- |
| If the CPL > 0.                                  |
| If CPUID.(EAX=07H, ECX=0H):EBX.SMAP[bit 20] = 0. |

## Real-Address Mode Exceptions

| #​​​UD                                           | If the LOCK prefix is used. |
| ------------------------------------------------ | --------------------------- |
| If CPUID.(EAX=07H, ECX=0H):EBX.SMAP[bit 20] = 0. |

## Virtual-8086 Mode Exceptions

| #​​​UD | The CLAC instruction is not recognized in virtual-8086 mode. |
| ------ | ------------------------------------------------------------ |

## Compatibility Mode Exceptions

| #​​​UD                                           | If the LOCK prefix is used. |
| ------------------------------------------------ | --------------------------- |
| If the CPL > 0.                                  |
| If CPUID.(EAX=07H, ECX=0H):EBX.SMAP[bit 20] = 0. |

## 64-Bit Mode Exceptions

| #​​​UD                                           | If the LOCK prefix is used. |
| ------------------------------------------------ | --------------------------- |
| If the CPL > 0.                                  |
| If CPUID.(EAX=07H, ECX=0H):EBX.SMAP[bit 20] = 0. |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
