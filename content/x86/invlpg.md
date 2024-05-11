#INVLPG
**Invalidate TLB Entries**

**Opcode1**

|         | Instruction | Op/En | 64-Bit Mode | Compat/Leg Mode | Description                                   |
| ------- | ----------- | ----- | ----------- | --------------- | --------------------------------------------- |
| 0F 01/7 |             |       | Valid       | Valid           | Invalidate TLB entries for page containing m. |

1. See the IA-32 Architecture Compatibility section below.

## Instruction Operand Encoding

| Op/En | Operand 1     | Operand 2 | Operand 3 | Operand 4 |
| ----- | ------------- | --------- | --------- | --------- |
| M     | ModRM:r/m (r) | N/A       | N/A       | N/A       |

## Description

Invalidates any translation lookaside buffer (TLB) entries specified with the source operand. The source operand is a memory address. The processor determines the page that contains that address and flushes all TLB entries for that page.1

The INVLPG instruction is a privileged instruction. When the processor is running in protected mode, the CPL must be 0 to execute this instruction.

The INVLPG instruction normally flushes TLB entries only for the specified page; however, in some cases, it may flush more entries, even the entire TLB. The instruction invalidates TLB entries associated with the current PCID and may or may not do so for TLB entries associated with other PCIDs. (If PCIDs are disabled — CR4.PCIDE = 0 — the current PCID is 000H.) The instruction also invalidates any global TLB entries for the specified page, regardless of PCID.

For more details on operations that flush the TLB, see “MOV—Move to/from Control Registers” in the Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 2B, and Section 4.10.4.1, “Operations that Invalidate TLBs and Paging-Structure Caches,” in the Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 3A.

This instruction’s operation is the same in all non-64-bit modes. It also operates the same in 64-bit mode, except if the memory address is in non-canonical form. In this case, INVLPG is the same as a NOP.

## IA-32 Architecture Compatibility

The INVLPG instruction is implementation dependent, and its function may be implemented differently on different families of Intel 64 or IA-32 processors. This instruction is not supported on IA-32 processors earlier than the Intel486 processor.

## Operation

```
Invalidate(RelevantTLBEntries);
Continue; (* Continue execution *)

```

## Flags Affected

None.

## Protected Mode Exceptions

| \#​​​​GP(0)                 | If the current privilege level is not 0. |
| --------------------------- | ---------------------------------------- |
| #​​​UD                      | Operand is a register.                   |
| If the LOCK prefix is used. |

> 1. If the paging structures map the linear address using a page larger than 4 KBytes and there are multiple TLB entries for that page (see Section 4.10.2.3, “Details of TLB Use,” in the Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 3A), the instruction invalidates all of them.

## Real-Address Mode Exceptions

| #​​​UD                      | Operand is a register. |
| --------------------------- | ---------------------- |
| If the LOCK prefix is used. |

## Virtual-8086 Mode Exceptions

| \#​​​​GP(0) | The INVLPG instruction cannot be executed at the virtual-8086 mode. |
| ----------- | ------------------------------------------------------------------- |

## 64-Bit Mode Exceptions

| \#​​​​GP(0)                 | If the current privilege level is not 0. |
| --------------------------- | ---------------------------------------- |
| #​​​UD                      | Operand is a register.                   |
| If the LOCK prefix is used. |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
