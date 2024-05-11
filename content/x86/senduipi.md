#SENDUIPI
**Send User Interprocessor Interrupt**

| Opcode/Instruction       | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                         |
| ------------------------ | ----- | ---------------------- | ------------------ | ----------------------------------- |
| F3 0F C7 /6 SENDUIPI reg | A     | V/I                    | UINTR              | Send interprocessor user interrupt. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1     | Operand 2 | Operand 3 | Operand 4 |
| ----- | ----- | ------------- | --------- | --------- | --------- |
| A     | N/A   | ModRM:reg (r) | N/A       | N/A       | N/A       |

## Description

The SENDUIPI instruction sends the user interprocessor interrupt (IPI) indicated by its register operand. (The operand always has 64 bits; operand-size overrides such as the prefix 66 are ignored.)

SENDUIPI uses a data structure called the user-interrupt target table (UITT). This table is located at the linear address UITTADDR (in the IA32_UINTR_TT MSR); it comprises UITTSZ+1 16-byte entries, where UITTSZ = IA32_UINT_MISC[31:0]. SENDUIPI uses the UITT entry (UITTE) indexed by the instruction's register operand. Each UITTE has the following format:

- Bit 0: V, a valid bit.
- Bits 7:1 are reserved and must be 0.
- Bits 15:8: UV, the user-interrupt vector (in the range 0–63, so bits 15:14 must be 0).
- Bits 63:16 are reserved.
- Bits 127:64: UPIDADDR, the linear address of a user posted-interrupt descriptor (UPID). (UPIDADDR is 64-byte aligned, so bits 69:64 of each UITTE must be 0.)

Each UPID has the following format (fields and bits not referenced are reserved):

- Bit 0 (ON) indicates an outstanding notification. If this bit is set, there is a notification outstanding for one or more user interrupts in PIR.
- Bit 1 (SN) indicates that notifications should be suppressed. If this bit is set, agents (including SENDUIPI) should not send notifications when posting user interrupts in this descriptor.
- Bits 23:16 (NV) contain the notification vector. This is used by agents sending user-interrupt notifications (including SENDUIPI).
- Bits 63:32 (NDST) contain the notification destination. This is the target physical APIC ID (in xAPIC mode, bits 47:40 are the 8-bit APIC ID; in x2APIC mode, the entire field forms the 32-bit APIC ID).
- Bits 127:64 (PIF) contain posted-interrupt requests. There is one bit for each user-interrupt vector. There is a user-interrupt request for a vector if the corresponding bit is 1.

Although SENDUIPI may be executed at any privilege level, all of the instruction’s memory accesses (to a UITTE and a UPID) are performed with supervisor privilege.

SENDUIPI sends a user interrupt by posting a user interrupt with vector V in the UPID referenced by UPIDADDR and then sending, as an ordinary IPI, any notification interrupt specified in that UPID.

## Operation

```
IF reg > UITTSZ;
    THEN #​​​​GP(0);
FI;
read tempUITTE from 16 bytes at UITTADDR+ (reg « 4);
IF tempUITTE.V = 0 or tempUITTE sets any reserved bit
    THEN #​​​​GP(0);
FI;
read tempUPID from 16 bytes at tempUITTE.UPIDADDR;// under lock
IF tempUPID sets any reserved bits or bits that must be zero
    THEN #​​​​GP(0); // release lock
FI;
tempUPID.PIR[tempUITTE.UV] := 1;
IF tempUPID.SN = tempUPID.ON = 0
    THEN
        tempUPID.ON := 1;
        sendNotify := 1;
    ELSE sendNotify := 0;
FI;
write tempUPID to 16 bytes at tempUITTE.UPIDADDR;// release lock
IF sendNotify = 1
    THEN
        IF local APIC is in x2APIC mode
            THEN send ordinary IPI with vector tempUPID.NV
                to 32-bit physical APIC ID tempUPID.NDST;
            ELSE send ordinary IPI with vector tempUPID.NV
                to 8-bit physical APIC ID tempUPID.NDST[15:8];
        FI;
FI;

```

## Flags Affected

None.

## Protected Mode Exceptions

| #​​​UD | The SENDUIPI instruction is not recognized in protected mode. |
| ------ | ------------------------------------------------------------- |

## Real-Address Mode Exceptions

| #​​​UD | The SENDUIPI instruction is not recognized in real-address mode. |
| ------ | ---------------------------------------------------------------- |

## Virtual-8086 Mode Exceptions

| #​​​UD | The SENDUIPI instruction is not recognized in virtual-8086 mode. |
| ------ | ---------------------------------------------------------------- |

## Compatibility Mode Exceptions

| #​​​UD | The SENDUIPI instruction is not recognized in compatibility mode. |
| ------ | ----------------------------------------------------------------- |

## 64-Bit Mode Exceptions

| #​​​UD                                                                                                                    | If the LOCK prefix is used.                          |
| ------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| If executed inside an enclave.                                                                                            |
| If CR4.UINTR = 0.                                                                                                         |
| If IA32_UINTR_TT[0] = 0.                                                                                                  |
| If CPUID.07H.0H:EDX.UINTR[bit 5] = 0.                                                                                     |
| \#​PF                                                                                                                     | If a page fault occurs.                              |
| \#​​​​GP                                                                                                                  | If the value of the register operand exceeds UITTSZ. |
| If the selected UITTE is not valid or sets any reserved bits.                                                             |
| If the selected UPID sets any reserved bits.                                                                              |
| If there is an attempt to access memory using a linear address that is not canonical relative to the current paging mode. |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
