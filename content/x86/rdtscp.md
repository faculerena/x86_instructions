# RDTSCP**Read Time**

| Opcode\* | Instruction | Op/En | 64-Bit Mode | Compat/Leg Mode | Description                                                                 |
| -------- | ----------- | ----- | ----------- | --------------- | --------------------------------------------------------------------------- |
| 0F 01 F9 | RDTSCP      | ZO    | Valid       | Valid           | Read 64-bit time-stamp counter and IA32_TSC_AUX value into EDX:EAX and ECX. |

## Instruction Operand Encoding

| Op/En | Operand 1 | Operand 2 | Operand 3 | Operand 4 |
| ----- | --------- | --------- | --------- | --------- |
| ZO    | N/A       | N/A       | N/A       | N/A       |

## Description

Reads the current value of the processor’s time-stamp counter (a 64-bit MSR) into the EDX:EAX registers and also reads the value of the IA32_TSC_AUX MSR (address C0000103H) into the ECX register. The EDX register is loaded with the high-order 32 bits of the IA32_TSC MSR; the EAX register is loaded with the low-order 32 bits of the IA32_TSC MSR; and the ECX register is loaded with the low-order 32-bits of IA32_TSC_AUX MSR. On processors that support the Intel 64 architecture, the high-order 32 bits of each of RAX, RDX, and RCX are cleared.

The processor monotonically increments the time-stamp counter MSR every clock cycle and resets it to 0 whenever the processor is reset. See “Time Stamp Counter” in Chapter 18 of the Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 3B, for specific details of the time stamp counter behavior.

The time stamp disable (TSD) flag in register CR4 restricts the use of the RDTSCP instruction as follows. When the flag is clear, the RDTSCP instruction can be executed at any privilege level; when the flag is set, the instruction can only be executed at privilege level 0.

The RDTSCP instruction is not a serializing instruction, but it does wait until all previous instructions have executed and all previous loads are globally visible.1 But it does not wait for previous stores to be globally visible, and subsequent instructions may begin execution before the read operation is performed. The following items may guide software seeking to order executions of RDTSCP:

- If software requires RDTSCP to be executed only after all previous stores are globally visible, it can execute MFENCE immediately before RDTSCP.
- If software requires RDTSCP to be executed prior to execution of any subsequent instruction (including any memory accesses), it can execute LFENCE immediately after RDTSCP.

See “Changes to Instruction Behavior in VMX Non-Root Operation” in Chapter 26 of the Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 3C, for more information about the behavior of this instruction in VMX non-root operation.

> 1. A load is considered to become globally visible when the value to be loaded is determined.

## Operation

```
IF (CR4.TSD = 0) or (CPL = 0) or (CR0.PE = 0)
    THEN
        EDX:EAX := TimeStampCounter;
        ECX := IA32_TSC_AUX[31:0];
    ELSE (* CR4.TSD = 1 and (CPL = 1, 2, or 3) and CR0.PE = 1 *)
        #​​​​GP(0);
FI;

```

## Flags Affected

None.

## Protected Mode Exceptions

| \#​​​​GP(0)                                | If the TSD flag in register CR4 is set and the CPL is greater than 0. |
| ------------------------------------------ | --------------------------------------------------------------------- |
| #​​​UD                                     | If the LOCK prefix is used.                                           |
| If CPUID.80000001H:EDX.RDTSCP[bit 27] = 0. |

## Real-Address Mode Exceptions

| #​​​UD                                     | If the LOCK prefix is used. |
| ------------------------------------------ | --------------------------- |
| If CPUID.80000001H:EDX.RDTSCP[bit 27] = 0. |

## Virtual-8086 Mode Exceptions

| \#​​​​GP(0)                                | If the TSD flag in register CR4 is set. |
| ------------------------------------------ | --------------------------------------- |
| #​​​UD                                     | If the LOCK prefix is used.             |
| If CPUID.80000001H:EDX.RDTSCP[bit 27] = 0. |

## Compatibility Mode Exceptions

Same exceptions as in protected mode.

## 64-Bit Mode Exceptions

Same exceptions as in protected mode.

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
