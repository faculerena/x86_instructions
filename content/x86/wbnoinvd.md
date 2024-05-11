#WBNOINVD
**Write Back and Do Not Invalidate Cache**

| Opcode / Instruction | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                                             |
| -------------------- | ----- | ---------------------- | ------------------ | ------------------------------------------------------------------------------------------------------- |
| F3 0F 09 WBNOINVD    | ZO    | V/V                    | WBNOINVD           | Write back and do not flush internal caches; initiate writing-back without flushing of external caches. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1 | Operand 2 | Operand 3 | Operand 4 |
| ----- | ----- | --------- | --------- | --------- | --------- |
| ZO    | N/A   | N/A       | N/A       | N/A       | N/A       |

## Description

The WBNOINVD instruction writes back all modified cache lines in the processor’s internal cache to main memory but does not invalidate (flush) the internal caches.

After executing this instruction, the processor does not wait for the external caches to complete their write-back operation before proceeding with instruction execution. It is the responsibility of hardware to respond to the cache write-back signal. The amount of time or cycles for WBNOINVD to complete will vary due to size and other factors of different cache hierarchies. As a consequence, the use of the WBNOINVD instruction can have an impact on logical processor interrupt/event response time.

The WBNOINVD instruction is a privileged instruction. When the processor is running in protected mode, the CPL of a program or procedure must be 0 to execute this instruction. This instruction is also a serializing instruction (see “Serializing Instructions” in Chapter 9 of the Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 3A).

This instruction’s operation is the same in non-64-bit modes and 64-bit mode.

## Operation

```
WriteBack(InternalCaches);
Continue; (* Continue execution *)

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
WBNOINVD void _wbnoinvd(void);

```

## Flags Affected

None.

## Protected Mode Exceptions

| \#​​​​GP(0) | If the current privilege level is not 0. |
| ----------- | ---------------------------------------- |
| #​​​UD      | If the LOCK prefix is used.              |

## Real-Address Mode Exceptions

| #​​​UD | If the LOCK prefix is used. |
| ------ | --------------------------- |

## Virtual-8086 Mode Exceptions

| \#​​​​GP(0) | WBNOINVD cannot be executed at the virtual-8086 mode. |
| ----------- | ----------------------------------------------------- |

## Compatibility Mode Exceptions

Same exceptions as in protected mode.

## 64-Bit Mode Exceptions

Same exceptions as in protected mode.

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
