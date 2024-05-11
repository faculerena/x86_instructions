# INVD

**Invalidate Internal Caches**

**Opcode1**

|       | Instruction | Op/En | 64-Bit Mode | Compat/Leg Mode | Description                                                  |
| ----- | ----------- | ----- | ----------- | --------------- | ------------------------------------------------------------ |
| 0F 08 | INVD        | ZO    | Valid       | Valid           | Flush internal caches; initiate flushing of external caches. |

> 1. See the IA-32 Architecture Compatibility section below.

## Instruction Operand Encoding

| Op/En | Operand 1 | Operand 2 | Operand 3 | Operand 4 |
| ----- | --------- | --------- | --------- | --------- |
| ZO    | N/A       | N/A       | N/A       | N/A       |

## Description

Invalidates (flushes) the processor’s internal caches and issues a special-function bus cycle that directs external caches to also flush themselves. Data held in internal caches is not written back to main memory.

After executing this instruction, the processor does not wait for the external caches to complete their flushing operation before proceeding with instruction execution. It is the responsibility of hardware to respond to the cache flush signal.

The INVD instruction is a privileged instruction. When the processor is running in protected mode, the CPL of a program or procedure must be 0 to execute this instruction.

The INVD instruction may be used when the cache is used as temporary memory and the cache contents need to be invalidated rather than written back to memory. When the cache is used as temporary memory, no external device should be actively writing data to main memory.

Use this instruction with care. Data cached internally and not written back to main memory will be lost. Note that any data from an external device to main memory (for example, via a PCIWrite) can be temporarily stored in the caches; these data can be lost when an INVD instruction is executed. Unless there is a specific requirement or benefit to flushing caches without writing back modified cache lines (for example, temporary memory, testing, or fault recovery where cache coherency with main memory is not a concern), software should instead use the WBINVD instruction.

On processors that support processor reserved memory, the INVD instruction cannot be executed when processor reserved memory protections are activated. See Section 36.5, “EPC and Management of EPC Pages,” in the Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 3D.

Some processors prevent execution of INVD after BIOS execution is complete. They report this by enumerating CPUID.(EAX=07H,ECX=1H):EAX[bit 30] as 1. On such processors, INVD cannot be executed if bit 0 of SR_BIOS_DONE (MSR address 151H) is 1.

This instruction’s operation is the same in non-64-bit modes and 64-bit mode.

## IA-32 Architecture Compatibility

The INVD instruction is implementation dependent; it may be implemented differently on different families of Intel 64 or IA-32 processors. This instruction is not supported on IA-32 processors earlier than the Intel486 processor.

## Operation

```
Flush(InternalCaches);
SignalFlush(ExternalCaches);
Continue (* Continue execution *)

```

## Flags Affected

None.

## Protected Mode Exceptions

| \#​​​​GP(0)                                                                                  | If the current privilege level is not 0. |
| -------------------------------------------------------------------------------------------- | ---------------------------------------- |
| If the processor reserved memory protections are activated.                                  |
| If CPUID.(EAX=07H, ECX=1H):EAX[30] = 1 and bit 0 is set in MSR_BIOS_DONE (MSR address 151H). |
| #​​​UD                                                                                       | If the LOCK prefix is used.              |

## Real-Address Mode Exceptions

| \#​​​​GP(0)                                                 | If CPUID.(EAX=07H, ECX=1H):EAX[30] = 1 and bit 0 is set in MSR_BIOS_DONE (MSR address 151H). |
| ----------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| If the processor reserved memory protections are activated. |
| #​​​UD                                                      | If the LOCK prefix is used.                                                                  |

## Virtual-8086 Mode Exceptions

| \#​​​​GP(0) | The INVD instruction cannot be executed in virtual-8086 mode. |
| ----------- | ------------------------------------------------------------- |

## Compatibility Mode Exceptions

Same exceptions as in protected mode.

## 64-Bit Mode Exceptions

Same exceptions as in protected mode.

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
