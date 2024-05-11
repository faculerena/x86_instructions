# ENQCMD**Enqueue Command**

| Opcode/Instruction                             | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                                                                                                   |
| ---------------------------------------------- | ----- | ---------------------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| F2 0F 38 F8 !(11):rrr:bbb ENQCMD r32/r64, m512 | A     | V/V                    | ENQCMD             | Atomically enqueue 64-byte user command from source memory operand to destination offset in ES segment specified in register operand as offset in ES segment. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1     | Operand 2     | Operand 3 | Operand 4 |
| ----- | ----- | ------------- | ------------- | --------- | --------- |
| A     | N/A   | ModRM:reg (w) | ModRM:r/m (r) | N/A       | N/A       |

### Description

The ENQCMD instruction allows software to write commands to **enqueue registers**, which are special device registers accessed using memory-mapped I/O (MMIO).

Enqueue registers expect writes to have the following format:

| 511 32 31 30 20 19                          |
| ------------------------------------------- |
| DEVICE SPECIFIC COMMAND PRIV RESERVED PASID |

[Figure 3-16](/x86/enqcmd#fig-3-16). 64-Byte Data Written to Enqueue Registers

Bits 19:0 convey the process address space identifier (PASID), a value which system software may assign to individual software threads. Bit 31 contains privilege identification (0 = user; 1 = supervisor). Devices implementing enqueue registers may use these two values along with a device-specific command in the upper 60 bytes.

The ENQCMD instruction begins by reading 64 bytes of command data from its source memory operand. This is an ordinary load with cacheability and memory ordering implied normally by the memory type. The source operand need not be aligned, and there is no guarantee that all 64 bytes are loaded atomically. Bits 31:0 of the source operand must be zero.

The instruction then formats those 64 bytes into **command data** with a format consistent with that given in [Figure 3-16](/x86/enqcmd#fig-3-16):

- Command[19:0] get IA32_PASID[19:0].1
- Command[30:20] are zero.
- Command[31] is 0 (indicating user; this value is used regardless of CPL).
- Command[511:32] get bits 511:32 of the source operand that was read from memory.

The ENQCMD instruction uses an **enqueue store** (defined below) to write this command data to the destination operand. The address of the destination operand is specified in a general-purpose register as an offset into the ES segment (the segment cannot be overridden).2 The destination linear address must be 64-byte aligned. The operation of an enqueue store disregards the memory type of the destination memory address.

> 1. ItisexpectedthatsystemsoftwarewillloadtheIA32_PASIDMSRsothatbits19:0containthePASIDofthecurrentsoft-ware thread. The MSR’s valid bit, IA32_PASID[31], must be 1. For additional details on the IA32_PASID MSR, see the Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 4.
> 2. In64-bitmode,thewidthoftheregisteroperandis64bits(32bitswitha67Hprefix).Outside64-bitmodewhenCS.D= 1, the width is 32 bits (16 bits with a 67H prefix). Outside 64-bit mode when CS.D=0, the width is 16 bits (32 bits with a 67H prefix).

An enqueue store is not ordered relative to older stores to WB or WC memory (including non-temporal stores) or to executions of the CLFLUSHOPT or CLWB (when applied to addresses other than that of the enqueue store). Software can enforce such ordering by executing a fencing instruction such as SFENCE or MFENCE before the enqueue store.

An enqueue store does not write the data into the cache hierarchy, nor does it fetch any data into the cache hierarchy. An enqueue store’s command data is never combined with that of any other store to the same address.

Unlike other stores, an enqueue store returns a status, which the ENQCMD instruction loads into the ZF flag in the RFLAGS register:

- ZF = 0 (success) reports that the 64-byte command data was written atomically to a device’s enqueue register and has been accepted by the device. (It does not guarantee that the device has acted on the command; it may have queued it for later execution.)
- ZF = 1 (retry) reports that the command data was not accepted. This status is returned if the destination address is an enqueue register but the command was not accepted due to capacity or other temporal reasons. This status is also returned if the destination address was not an enqueue register (including the case of a memory address); in these cases, the store is dropped and is written neither to MMIO nor to memory.

Availability of the ENQCMD instruction is indicated by the presence of the CPUID feature flag ENQCMD (CPUID.(EAX=07H, ECX=0H):ECX[bit 29]).

### Operation

```
IF IA32_PASID[31] = 0
    THEN #​​​​GP;
ELSE
    COMMAND := (SRC & ~FFFFFFFFH) | (IA32_PASID & FFFFFH);
    DEST := COMMAND;
FI;

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
ENQCMD int_enqcmd(void *dst, const void *src)

```

### Flags Affected

The ZF flag is set if the enqueue-store completion returns the retry status; otherwise it is cleared. All other flags are cleared.

### SIMD Floating-Point Exceptions

None.

### Protected Mode Exceptions

| \#​​​​GP(0)                                                         | For an illegal memory operand effective address in the CS, DS, ES, FS or GS segments. |
| ------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| If destination linear address is not aligned to a 64-byte boundary. |
| If the PASID Valid field (bit 31) is 0 in IA32_PASID MSR.           |
| If bits 31:0 of the source operand are not all zero.                |
| \#​​​​​SS(0)                                                        | For an illegal address in the SS segment.                                             |
| \#​PF(fault-code)                                                   | For a page fault.                                                                     |
| #​​​UD                                                              | If CPUID.07H.0H:ECX.ENQCMD[bit 29] = 0.                                               |
| If the LOCK prefix is used.                                         |

### Real-Address Mode Exceptions

| \#​​​​GP                                                            | If any part of the operand lies outside the effective address space from 0 to FFFFH. |
| ------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| If destination linear address is not aligned to a 64-byte boundary. |
| If the PASID Valid field (bit 31) is 0 in IA32_PASID MSR.           |
| If bits 31:0 of the source operand are not all zero.                |
| #​​​UD                                                              | If CPUID.07H.0H:ECX.ENQCMD[bit 29] = 0.                                              |
| If the LOCK prefix is used.                                         |

### Virtual-8086 Mode Exceptions

Same exceptions as in real-address mode. Additionally:

| \#​PF(fault-code) | For a page fault. |
| ----------------- | ----------------- |

### Compatibility Mode Exceptions

Same exceptions as in protected mode.

### 64-Bit Mode Exceptions

| \#​​​​​SS(0)                                                        | If a memory address referencing the SS segment is in non-canonical form. |
| ------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| \#​​​​GP(0)                                                         | If the memory address is in non-canonical form.                          |
| If destination linear address is not aligned to a 64-byte boundary. |
| If the PASID Valid field (bit 31) is 0 in IA32_PASID MSR.           |
| If bits 31:0 of the source operand are not all zero.                |
| \#​PF(fault-code)                                                   | For a page fault.                                                        |
| #​​​UD                                                              | If CPUID.07H.0H:ECX.ENQCMD[bit 29].                                      |
| If the LOCK prefix is used.                                         |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
