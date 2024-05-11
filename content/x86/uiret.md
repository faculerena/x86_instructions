#UIRET
**User**

| Opcode/Instruction | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                            |
| ------------------ | ----- | ---------------------- | ------------------ | -------------------------------------- |
| F3 0F 01 EC UIRET  | ZO    | V/I                    | UINTR              | Return from handling a user interrupt. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1 | Operand 2 | Operand 3 | Operand 4 |
| ----- | ----- | --------- | --------- | --------- | --------- |
| ZO    | N/A   | N/A       | N/A       | N/A       | N/A       |

## Description

UIRET returns from the handling of a user interrupt. It can be executed regardless of CPL.

Execution of UIRET inside a transactional region causes a transactional abort; the abort loads EAX as it would have had it been due to an execution of IRET.

UIRET can be tracked by Architectural Last Branch Records (LBRs), Intel Processor Trace (Intel PT), and Performance Monitoring. For both Intel PT and LBRs, UIRET is recorded in precisely the same manner as IRET. Hence for LBRs, UIRETs fall into the OTHER_BRANCH category, which implies that IA32_LBR_CTL.OTHER_BRANCH[bit 22] must be set to record user-interrupt delivery, and that the IA32_LBR_x_INFO.BR_TYPE field will indicate OTHER_BRANCH for any recorded user interrupt. For Intel PT, control flow tracing must be enabled by setting IA32_RTIT_CTL.BranchEn[bit 13].

UIRET will also increment performance counters for which counting BR_INST_RETIRED.FAR_BRANCH is enabled.

## Operation

```
Pop tempRIP;
Pop tempRFLAGS; // see below for how this is used to load RFLAGS
Pop tempRSP;
IF tempRIP is not canonical in current paging mode
    THEN #​​​​GP(0);
FI;
IF ShadowStackEnabled(CPL)
    THEN
        PopShadowStack SSRIP;
        IF SSRIP ≠ tempRIP
            THEN #​CP (FAR-RET/IRET);
        FI;
FI;
RIP := tempRIP;
// update in RFLAGS only CF, PF, AF, ZF, SF, TF, DF, OF, NT, RF, AC, and ID
RFLAGS := (RFLAGS & ~254DD5H) | (tempRFLAGS & 254DD5H);
RSP := tempRSP;
UIF := 1;
Clear any cache-line monitoring established by MONITOR or UMONITOR;

```

## Flags Affected

See the Operation section.

## Protected Mode Exceptions

| #​​​UD | The UIRET instruction is not recognized in protected mode. |
| ------ | ---------------------------------------------------------- |

## Real-Address Mode Exceptions

| #​​​UD | The UIRET instruction is not recognized in real-address mode. |
| ------ | ------------------------------------------------------------- |

## Virtual-8086 Mode Exceptions

| #​​​UD | The UIRET instruction is not recognized in virtual-8086 mode. |
| ------ | ------------------------------------------------------------- |

## Compatibility Mode Exceptions

| #​​​UD | The UIRET instruction is not recognized in compatibility mode. |
| ------ | -------------------------------------------------------------- |

## 64-Bit Mode Exceptions

| \#​​​​GP(0)                           | If the return instruction pointer is non-canonical.                                                                |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| \#​​​​​SS(0)                          | If an attempt to pop a value off the stack causes a non-canonical address to be referenced.                        |
| \#​PF(fault-code)                     | If a page fault occurs.                                                                                            |
| \#​AC(0)                              | If alignment checking is enabled and an unaligned memory reference is made while the current privilege level is 3. |
| \#​CP                                 | If return instruction pointer from stack and shadow stack do not match.                                            |
| #​​​UD                                | If the LOCK prefix is used.                                                                                        |
| If executed inside an enclave.        |
| If CR4.UINTR = 0.                     |
| If CPUID.07H.0H:EDX.UINTR[bit 5] = 0. |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
