#CLRSSBSY
**Clear Busy Flag in a Supervisor Shadow Stack Token**

| Opcode/Instruction       | Op / En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                        |
| ------------------------ | ------- | ---------------------- | ------------------ | ------------------------------------------------------------------ |
| F3 0F AE /6 CLRSSBSY m64 | M       | V/V                    | CET_SS             | Clear busy flag in supervisor shadow stack token reference by m64. |

## Instruction Operand Encoding

| Op/En | Tuple Type | Operand 1        | Operand 2 | Operand 3 | Operand 4 |
| ----- | ---------- | ---------------- | --------- | --------- | --------- |
| M     | N/A        | ModRM:r/m (r, w) | N/A       | N/A       | N/A       |

## Description

Clear busy flag in supervisor shadow stack token reference by m64. Subsequent to marking the shadow stack as not busy the SSP is loaded with value 0.

## Operation

```
IF (CR4.CET = 0)
    THEN #​​​UD; FI;
IF (IA32_S_CET.SH_STK_EN = 0)
    THEN #​​​UD; FI;
IF CPL > 0
    THEN GP(0); FI;
SSP_LA = Linear_Address(mem operand)
IF SSP_LA not aligned to 8 bytes
    THEN #​​​​GP(0); FI;
expected_token_value=SSP_LA|BUSY_BIT (*busybit-bitposition0-mustbeset*)
new_token_value = SSP_LA (* Clear the busy bit *)
IF shadow_stack_lock_cmpxchg8b(SSP_LA, new_token_value, expected_token_value) != expected_token_value
    invalid_token := 1; FI
(* Set the CF if invalid token was detected *)
RFLAGS.CF = (invalid_token == 1) ? 1 : 0;
RFLAGS.ZF,PF,AF,OF,SF := 0;
SSP := 0

```

## Flags Affected

CF is set if an invalid token was detected, else it is cleared. ZF, PF, AF, OF, and SF are cleared.

## Protected Mode Exceptions

| #​​​UD                                                                                              | If the LOCK prefix is used.                                            |
| --------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| If CR4.CET = 0.                                                                                     |
| IF IA32_S_CET.SH_STK_EN = 0.                                                                        |
| \#​​​​GP(0)                                                                                         | If memory operand linear address not aligned to 8 bytes.               |
| If a memory operand effective address is outside the CS, DS, ES, FS, or GS segment limit.           |
| If destination is located in a non-writeable segment.                                               |
| If the DS, ES, FS, or GS register is used to access memory and it contains a NULL segment selector. |
| If CPL is not 0.                                                                                    |
| \#​​​​​SS(0)                                                                                        | If a memory operand effective address is outside the SS segment limit. |
| \#​PF(fault-code)                                                                                   | If a page fault occurs.                                                |

## Real-Address Mode Exceptions

| #​​​UD | The CLRSSBSY instruction is not recognized in real-address mode. |
| ------ | ---------------------------------------------------------------- |

## Virtual-8086 Mode Exceptions

| #​​​UD | The CLRSSBSY instruction is not recognized in virtual-8086 mode. |
| ------ | ---------------------------------------------------------------- |

## Compatibility Mode Exceptions

| #​​​UD            | Same exceptions as in protected mode. |
| ----------------- | ------------------------------------- |
| \#​​​​GP(0)       | Same exceptions as in protected mode. |
| \#​PF(fault-code) | If a page fault occurs.               |

## 64-Bit Mode Exceptions

| #​​​UD                                            | If the LOCK prefix is used.                                                |
| ------------------------------------------------- | -------------------------------------------------------------------------- |
| If CR4.CET = 0.                                   |
| IF IA32_S_CET.SH_STK_EN = 0.                      |
| \#​​​​GP(0)                                       | If memory operand linear address not aligned to 8 bytes.                   |
| If CPL is not 0.                                  |
| If the memory address is in a non-canonical form. |
| If token is invalid.                              |
| \#​​​​​SS(0)                                      | If a memory address referencing the SS segment is in a non-canonical form. |
| \#​PF(fault-code)                                 | If a page fault occurs.                                                    |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
