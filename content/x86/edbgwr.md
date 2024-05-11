# EDBGWR

**Write to a Debug Enclave**

| Opcode/Instruction      | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                    |
| ----------------------- | ----- | ---------------------- | ------------------ | -------------------------------------------------------------- |
| EAX = 05H ENCLS[EDBGWR] | IR    | V/V                    | SGX1               | This leaf function writes a dword/quadword to a debug enclave. |

## Instruction Operand Encoding

| Op/En | EAX         |                         | RBX                                        | RCX                                      |
| ----- | ----------- | ----------------------- | ------------------------------------------ | ---------------------------------------- |
| IR    | EDBGWR (In) | Return error code (Out) | Data to be written to a debug enclave (In) | Address of Target memory in the EPC (In) |

### Description

This leaf function copies the content in EBX/RBX to an EPC page belonging to a debug enclave. Eight bytes are written in 64-bit mode, four bytes are written in non-64-bit modes. The size of data cannot be overridden.

The effective address of the target location inside the EPC is provided in the register RCX.

## EDBGWR Memory Parameter Semantics

| EPCQW                             |
| --------------------------------- |
| Write access permitted by Enclave |

The instruction faults if any of the following:

## EDBGWR Faulting Conditions

| RCX points into a page that is an SECS.                                        | RCX does not resolve to a naturally aligned linear address.       |
| ------------------------------------------------------------------------------ | ----------------------------------------------------------------- |
| RCX points to a page that does not belong to an enclave that is in debug mode. | RCX points to a location inside a TCS that is not the FLAGS word. |
| An operand causing any segment violation.                                      | May page fault.                                                   |
| CPL > 0.                                                                       |                                                                   |

The error codes are:

| Error Code (see Table 38-4) | Description                                                                     |
| --------------------------- | ------------------------------------------------------------------------------- |
| No Error                    | EDBGWR successful.                                                              |
| SGX_PAGE_NOT_DEBUGGABLE     | The EPC page cannot be accessed because it is in the PENDING or MODIFIED state. |

Table 38-20. EDBGWR Return Value in RAX

This instruction ignores the EPCM RWX attributes on the enclave page. Consequently, violation of EPCM RWX attributes via EDBGRD does not result in a #​​​​GP.

### Concurrency Restrictions

| Leaf                                                        | Parameter       | Base Concurrency Restrictions |     |     |
| ----------------------------------------------------------- | --------------- | ----------------------------- | --- | --- |
|                                                             | On Conflict     |                               |
| EDBGWR EDBGWR Target [DS:RCX] Shared EDBGWR Target [DS:RCX] | Target [DS:RCX] |                               |     |     |

Table 38-21. Base Concurrency Restrictions of EDBGWR

| Leaf                                                                                                                                                                                     | Parameter       | Additional Concurrency Restrictions                                   |     |                     |     |            |     |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------- | --------------------------------------------------------------------- | --- | ------------------- | --- | ---------- | --- |
| vs. EACCEPT, EACCEPTCOPY, vs. EADD, EEXTEND, EINIT vs. ETRACK, ETRACKC Access vs. ETRACK, ETRACKC Access On Conflict Access vs. ETRACK, ETRACKC Access On Conflict EMODPE, EMODPR, EMODT |                 | vs. EADD, EEXTEND, EINIT vs. EADD, EEXTEND, EINIT vs. ETRACK, ETRACKC |     | vs. ETRACK, ETRACKC |     |
| Access On Conflict Access On Conflict Access Access On Conflict Access On Conflict                                                                                                       |                 |                                                                       |     |                     |     |
| EDBGWR                                                                                                                                                                                   | Target [DS:RCX] | Concurrent                                                            |     | Concurrent          |     | Concurrent |     |

Table 38-22. Additional Concurrency Restrictions of EDBGWR

### Operation

## Temp Variables in EDBGWR Operational Flow

| Name       | Type   | Size (Bits) | Description                                                              |
| ---------- | ------ | ----------- | ------------------------------------------------------------------------ |
| TMP_MODE64 | Binary | 1           | ((IA32_EFER.LMA = 1) && (CS.L = 1)).                                     |
| TMP_SECS   |        | 64          | Physical address of SECS of the enclave to which source operand belongs. |

TMP_MODE64 := ((IA32_EFER.LMA = 1) && (CS.L = 1));

IF ( (TMP_MODE64 = 1) and (DS:RCX is not 8Byte Aligned) )

THEN #​​​​GP(0); FI;

IF ( (TMP_MODE64 = 0) and (DS:RCX is not 4Byte Aligned) )

THEN #​​​​GP(0); FI;

IF (DS:RCX does not resolve within an EPC)

THEN #​PF(DS:RCX); FI;

(\* make sure no other Intel SGX instruction is accessing the same EPCM entry \*)

IF (Another instruction modifying the same EPCM entry is executing)

THEN #​​​​GP(0); FI;

IF (EPCM(DS:RCX).VALID = 0)

THEN #​PF(DS:RCX); FI;

(\* make sure that DS:RCX (DST) is pointing to a PT_REG or PT_TCS or PT_SS_FIRST or PT_SS_REST \*)

IF ( (EPCM(DS:RCX).PT ≠ PT_REG) and (EPCM(DS:RCX).PT ≠ PT_TCS)

and (EPCM(DS:RCX).PT ≠ PT_SS_FIRST) and (EPCM(DS:RCX).PT ≠ PT_SS_REST))

THEN #​PF(DS:RCX); FI;

(\* make sure that DS:RCX points to an accessible EPC page \*)

IF ( (EPCM(DS:RCX).PENDING is not 0) or (EPCM(DS:RCS).MODIFIED is not 0) )

THEN

RFLAGS.ZF := 1;

RAX := SGX_PAGE_NOT_DEBUGGABLE;

GOTO DONE;

FI;

(\* If destination is a TCS, then make sure that the offset into the page can only point to the FLAGS field\*)

IF ( ( EPCM(DS:RCX). PT = PT_TCS) and ((DS:RCX) & FF8H ≠ offset_of_FLAGS & 0FF8H) )

THEN #​​​​GP(0); FI;

(\* Locate the SECS for the enclave to which the DS:RCX page belongs \*)

TMP_SECS := GET_SECS_PHYS_ADDRESS(EPCM(DS:RCX).ENCLAVESECS);

(\* make sure the enclave owning the PT_REG or PT_TCS page allow debug \*)

IF (TMP_SECS.ATTRIBUTES.DEBUG = 0)

THEN #​​​​GP(0); FI;

IF ( (TMP_MODE64 = 1) )

THEN (DS:RCX)[63:0] := RBX[63:0];

ELSE (DS:RCX)[31:0] := EBX[31:0];

FI;

(\* clear EAX and ZF to indicate successful completion \*)

RAX := 0;

RFLAGS.ZF := 0;

DONE:

(\* clear flags \*)

RFLAGS.CF,PF,AF,OF,SF := 0

### Flags Affected

ZF is set if the page is MODIFIED or PENDING; RAX contains the error code. Otherwise ZF is cleared and RAX is set to 0. CF, PF, AF, OF, SF are cleared.

### Protected Mode Exceptions

| \#​​​​GP(0)                                                                       | If the address in RCS violates DS limit or access rights.  |
| --------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| If DS segment is unusable.                                                        |
| If RCX points to a memory location not 4Byte-aligned.                             |
| If the address in RCX points to a page belonging to a non-debug enclave.          |
| If the address in RCX points to a page which is not PT_TCS or PT_REG.             |
| If the address in RCX points to a location inside TCS that is not the FLAGS word. |
| \#​PF(error                                                                       | code) If a page fault occurs in accessing memory operands. |
| If the address in RCX points to a non-EPC page.                                   |
| If the address in RCX points to an invalid EPC page.                              |

### 64-Bit Mode Exceptions

| \#​​​​GP(0)                                                                       | If RCX is non-canonical form.                              |
| --------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| If RCX points to a memory location not 8Byte-aligned.                             |
| If the address in RCX points to a page belonging to a non-debug enclave.          |
| If the address in RCX points to a page which is not PT_TCS or PT_REG.             |
| If the address in RCX points to a location inside TCS that is not the FLAGS word. |
| \#​PF(error                                                                       | code) If a page fault occurs in accessing memory operands. |
| If the address in RCX points to a non-EPC page.                                   |
| If the address in RCX points to an invalid EPC page.                              |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
