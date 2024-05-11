#SETSSBSY
**Mark Shadow Stack Busy**

| Opcode/Instruction   | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                               |
| -------------------- | ----- | ---------------------- | ------------------ | ------------------------------------------------------------------------- |
| F3 0F 01 E8 SETSSBSY | ZO    | V/V                    | CET_SS             | Set busy flag in supervisor shadow stack token reference by IA32_PL0_SSP. |

## Instruction Operand Encoding

| Op/En | Operand 1 | Operand 2 | Operand 3 | Operand 4 |
| ----- | --------- | --------- | --------- | --------- |
| ZO    | N/A       | N/A       | N/A       | N/A       |

## Description

The SETSSBSY instruction verifies the presence of a non-busy supervisor shadow stack token at the address in the IA32_PL0_SSP MSR and marks it busy. Following successful execution of the instruction, the SSP is set to the value of the IA32_PL0_SSP MSR.

## Operation

```
IF (CR4.CET = 0)
    THEN #​​​UD; FI;
IF (IA32_S_CET.SH_STK_EN = 0)
    THEN #​​​UD; FI;
IF CPL > 0
    THEN GP(0); FI;
SSP_LA = IA32_PL0_SSP
If SSP_LA not aligned to 8 bytes
    THEN #​​​​GP(0); FI;
expected_token_value = SSP_LA
new_token_value = SSP_LA | BUSY_BIT
IF shadow_stack_lock_cmpxchg8B(SSP_LA, new_token_value, expected_token_value) != expected_token_value
    THEN #​CP(SETSSBSY); FI;
SSP = SSP_LA

```

## Flags Affected

None.

## C/C++ Compiler Intrinsic Equivalent

```
SETSSBSYvoid _setssbsy(void);

```

## Protected Mode Exceptions

| #​​​UD                                                                        | If the LOCK prefix is used.             |
| ----------------------------------------------------------------------------- | --------------------------------------- |
| If CR4.CET = 0.                                                               |
| IF IA32_S_CET.SH_STK_EN = 0.                                                  |
| \#​​​​GP(0)                                                                   | If IA32_PL0_SSP not aligned to 8 bytes. |
| If CPL is not 0.                                                              |
| \#​CP(setssbsy)                                                               | If busy bit in token is set.            |
| If in 32-bit or compatibility mode, and the address in token is not below 4G. |
| \#​PF(fault-code)                                                             | If a page fault occurs.                 |

## Real-Address Mode Exceptions

| #​​​UD | The SETSSBSY instruction is not recognized in real-address mode. |
| ------ | ---------------------------------------------------------------- |

## Virtual-8086 Mode Exceptions

| #​​​UD | The SETSSBSY instruction is not recognized in virtual-8086 mode. |
| ------ | ---------------------------------------------------------------- |

## Compatibility Mode Exceptions

Same as protected mode exceptions.

## 64-Bit Mode Exceptions

Same as protected mode exceptions.

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
