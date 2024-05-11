# SAVEPREVSSP**Save Previous Shadow Stack Pointer**

| Opcode/Instruction                            | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                 |
| --------------------------------------------- | ----- | ---------------------- | ------------------ | ----------------------------------------------------------- |
| F3 0F 01 EA (mod!=11, /5, RM=010) SAVEPREVSSP | ZO    | V/V                    | CET_SS             | Save a restore-shadow-stack token on previous shadow stack. |

## Instruction Operand Encoding

| Op/En | Operand 1 | Operand 2 | Operand 3 | Operand 4 |
| ----- | --------- | --------- | --------- | --------- |
| ZO    | N/A       | N/A       | N/A       | N/A       |

## Description

Push a restore-shadow-stack token on the previous shadow stack at the next 8 byte aligned boundary. The previous SSP is obtained from the previous-ssp token at the top of the current shadow stack.

## Operation

```
IF CPL = 3
    IF (CR4.CET & IA32_U_CET.SH_STK_EN) = 0
        THEN #​​​UD; FI;
ELSE
    IF (CR4.CET & IA32_S_CET.SH_STK_EN) = 0
        THEN #​​​UD; FI;
FI;
IF SSP not aligned to 8 bytes
    THEN #​​​​GP(0); FI;
(* Pop the “previous-ssp” token from current shadow stack *)
previous_ssp_token = ShadowStackPop8B(SSP)
(* If the CF flag indicates there was a alignment hole on current shadow stack then pop that alignment hole *)
(* Note that the alignment hole must be zero and can be present only when in legacy/compatibility mode *)
IF RFLAGS.CF == 1 AND (IA32_EFER.LMA AND CS.L)
    #​​​​GP(0)
FI;
IF RFLAGS.CF == 1
    must_be_zero = ShadowStackPop4B(SSP)
    IF must_be_zero != 0 THEN #​​​​GP(0)
FI;
(* Previous SSP token must have the bit 1 set *)
IF ((previous_ssp_token & 0x02) == 0)
    THEN #​​​​GP(0); (* bit 1 was 0 *)
IF ((IA32_EFER.LMA AND CS.L) = 0 AND previous_ssp_token [63:32] != 0)
THEN #​​​​GP(0); FI; (* If compatibility/legacy mode and SSP not in 4G *)
(* Save Prev SSP from previous_ssp_token to the old shadow stack at next 8 byte aligned address *)
old_SSP = previous_ssp_token & ~0x03
temp := (old_SSP | (IA32_EFER.LMA & CS.L));
Shadow_stack_store 4 bytes of 0 to (old_SSP - 4)
old_SSP := old_SSP & ~0x07;
Shadow_stack_store 8 bytes of temp to (old_SSP - 8)

```

## Flags Affected

None.

## C/C++ Compiler Intrinsic Equivalent

```
SAVEPREVSSP void _saveprevssp(void);

```

## Protected Mode Exceptions

| #​​​UD                                                                               | If the LOCK prefix is used. |
| ------------------------------------------------------------------------------------ | --------------------------- |
| If CR4.CET = 0.                                                                      |
| IF CPL = 3 and IA32_U_CET.SH_STK_EN = 0.                                             |
| IF CPL < 3 and IA32_S_CET.SH_STK_EN = 0.                                             |
| \#​​​​GP(0)                                                                          | If SSP not 8 byte aligned.  |
| If alignment hole on shadow stack is not 0.                                          |
| If bit 1 of the previous-ssp token is not set to 1.                                  |
| If in 32-bit/compatibility mode and SSP recorded in previous-ssp token is beyond 4G. |
| \#​PF(fault-code)                                                                    | If a page fault occurs.     |

## Real-Address Mode Exceptions

| #​​​UD | The SAVEPREVSSP instruction is not recognized in real-address mode. |
| ------ | ------------------------------------------------------------------- |

## Virtual-8086 Mode Exceptions

| #​​​UD | The SAVEPREVSSP instruction is not recognized in virtual-8086 mode. |
| ------ | ------------------------------------------------------------------- |

## Compatibility Mode Exceptions

Same as protected mode exceptions.

## 64-Bit Mode Exceptions

| #​​​UD                                              | If the LOCK prefix is used. |
| --------------------------------------------------- | --------------------------- |
| If CR4.CET = 0.                                     |
| If CPL = 3 and IA32_U_CET.SH_STK_EN = 0.            |
| If CPL < 3 and IA32_S_CET.SH_STK_EN = 0.            |
| \#​​​​GP(0)                                         | If SSP not 8 byte aligned.  |
| If carry flag is set.                               |
| If bit 1 of the previous-ssp token is not set to 1. |
| \#​PF(fault-code)                                   | If a page fault occurs.     |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
