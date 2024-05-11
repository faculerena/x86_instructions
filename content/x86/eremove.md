# EREMOVE**Remove a page from the EPC**

| Opcode/Instruction       | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                     |
| ------------------------ | ----- | ---------------------- | ------------------ | ----------------------------------------------- |
| EAX = 03H ENCLS[EREMOVE] | IR    | V/V                    | SGX1               | This leaf function removes a page from the EPC. |

## Instruction Operand Encoding

| Op/En | EAX          |                         | RCX                                    |
| ----- | ------------ | ----------------------- | -------------------------------------- |
| IR    | EREMOVE (In) | Return error code (Out) | Effective address of the EPC page (In) |

### Description

This leaf function causes an EPC page to be un-associated with its SECS and be marked as unused. This instruction leaf can only be executed when the current privilege level is 0.

The content of RCX is an effective address of an EPC page. The DS segment is used to create linear address. Segment override is not supported.

The instruction fails if the operand is not properly aligned or does not refer to an EPC page or the page is in use by another thread, or other threads are running in the enclave to which the page belongs. In addition the instruction fails if the operand refers to an SECS with associations.

## EREMOVE Memory Parameter Semantics

| EPCPAGE                           |
| --------------------------------- |
| Write access permitted by Enclave |

The instruction faults if any of the following:

## EREMOVE Faulting Conditions

| The memory operand is not properly aligned.              | The memory operand does not resolve in an EPC page.       |
| -------------------------------------------------------- | --------------------------------------------------------- |
| Refers to an invalid SECS.                               | Refers to an EPC page that is locked by another thread.   |
| Another Intel SGX instruction is accessing the EPC page. | RCX does not contain an effective address of an EPC page. |
| the EPC page refers to an SECS with associations.        |                                                           |

The error codes are:

| Error Code (see Table 38-4) | Description                                                         |
| --------------------------- | ------------------------------------------------------------------- |
| No Error                    | EREMOVE successful.                                                 |
| SGX_CHILD_PRESENT           | If the SECS still have enclave pages loaded into EPC.               |
| SGX_ENCLAVE_ACT             | If there are still logical processors executing inside the enclave. |

Table 38-42. EREMOVE Return Value in RAX

### Concurrency Restrictions

| Leaf                                                                      | Parameter       | Base Concurrency Restrictions |     |     |
| ------------------------------------------------------------------------- | --------------- | ----------------------------- | --- | --- |
|                                                                           | On Conflict     |                               |
| EREMOVE EREMOVE Target [DS:RCX] Exclusive #​​​​GP EREMOVE Target [DS:RCX] | Target [DS:RCX] |                               |     |     |

Table 38-43. Base Concurrency Restrictions of EREMOVE

| Leaf                                                                                                                                                                                     | Parameter       | Additional Concurrency Restrictions                                   |     |                     |     |            |     |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------- | --------------------------------------------------------------------- | --- | ------------------- | --- | ---------- | --- |
| vs. EACCEPT, EACCEPTCOPY, vs. EADD, EEXTEND, EINIT vs. ETRACK, ETRACKC Access vs. ETRACK, ETRACKC Access On Conflict Access vs. ETRACK, ETRACKC Access On Conflict EMODPE, EMODPR, EMODT |                 | vs. EADD, EEXTEND, EINIT vs. EADD, EEXTEND, EINIT vs. ETRACK, ETRACKC |     | vs. ETRACK, ETRACKC |     |
| Access On Conflict Access On Conflict Access Access On Conflict Access On Conflict                                                                                                       |                 |                                                                       |     |                     |     |
| EREMOVE                                                                                                                                                                                  | Target [DS:RCX] | Concurrent                                                            |     | Concurrent          |     | Concurrent |     |

Table 38-44. Additional Concurrency Restrictions of EREMOVE

### Operation

## Temp Variables in EREMOVE Operational Flow

| Name     | Type              | Size (Bits) | Description                                     |
| -------- | ----------------- | ----------- | ----------------------------------------------- |
| TMP_SECS | Effective Address | 32/64       | Effective address of the SECS destination page. |

IF (DS:RCX is not 4KByte Aligned)

THEN #​​​​GP(0); FI;

IF (DS:RCX does not resolve to an EPC page)

THEN #​PF(DS:RCX); FI;

TMP_SECS := Get_SECS_ADDRESS();

(\* Check the EPC page for concurrency \*)

IF (EPC page being referenced by another Intel SGX instruction)

THEN

IF (<<VMX non-root operation>> AND <<ENABLE_EPC_VIRTUALIZATION_EXTENSIONS>>)

THEN

VMCS.Exit_reason := SGX_CONFLICT;

VMCS.Exit_qualification.code := EPC_PAGE_CONFLICT_EXCEPTION;

VMCS.Exit_qualification.error := 0;

VMCS.Guest-physical_address := << translation of DS:RCX produced by paging >>;

VMCS.Guest-linear_address := DS:RCX;

Deliver VMEXIT;

ELSE

#​​​​GP(0);

FI;

FI;

(\* if DS:RCX is already unused, nothing to do\*)

IF ( (EPCM(DS:RCX).VALID = 0) or (EPCM(DS:RCX).PT = PT_TRIM AND EPCM(DS:RCX).MODIFIED = 0))

THEN GOTO DONE;

FI;

IF ( (EPCM(DS:RCX).PT = PT_VA) OR

((EPCM(DS:RCX).PT = PT_TRIM) AND (EPCM(DS:RCX).MODIFIED = 0)) )

THEN

EPCM(DS:RCX).VALID := 0;

GOTO DONE;

FI;

IF (EPCM(DS:RCX).PT = PT_SECS)

THEN

IF (DS:RCX has an EPC page associated with it)

THEN

RFLAGS.ZF := 1;

RAX := SGX_CHILD_PRESENT;

GOTO ERROR_EXIT;

FI;

(\* treat SECS as having a child page when VIRTCHILDCNT is non-zero \*)

IF (<<in VMX non-root operation>> AND

<<ENABLE_EPC_VIRTUALIZATION_EXTENSIONS>> AND

(SECS(DS:RCX).VIRTCHILDCNT ≠ 0))

THEN

RFLAGS.ZF := 1;

RAX := SGX_CHILD_PRESENT

GOTO ERROR_EXIT

FI;

EPCM(DS:RCX).VALID := 0;

GOTO DONE;

FI;

IF (Other threads active using SECS)

THEN

RFLAGS.ZF := 1;

RAX := SGX_ENCLAVE_ACT;

GOTO ERROR_EXIT;

FI;

IF ( (EPCM(DS:RCX).PT is PT_REG) or (EPCM(DS:RCX).PT is PT_TCS) or (EPCM(DS:RCX).PT is PT_TRIM) or

(EPCM(DS:RCX).PT is PT_SS_FIRST) or (EPCM(DS:RCX).PT is PT_SS_REST))

THEN

EPCM(DS:RCX).VALID := 0;

GOTO DONE;

FI;

DONE:

RAX := 0;

RFLAGS.ZF := 0;

ERROR_EXIT:

RFLAGS.CF,PF,AF,OF,SF := 0;

### Flags Affected

Sets ZF if unsuccessful, otherwise cleared and RAX returns error code. Clears CF, PF, AF, OF, SF.

### Protected Mode Exceptions

| \#​​​​GP(0)                                             | If a memory operand effective address is outside the DS segment limit. |
| ------------------------------------------------------- | ---------------------------------------------------------------------- |
| If a memory operand is not properly aligned.            |
| If another Intel SGX instruction is accessing the page. |
| \#​PF(error                                             | code) If a page fault occurs in accessing memory operands.             |
| If the memory operand is not an EPC page.               |

### 64-Bit Mode Exceptions

| \#​​​​GP(0)                                             | If the memory operand is non-canonical form.               |
| ------------------------------------------------------- | ---------------------------------------------------------- |
| If a memory operand is not properly aligned.            |
| If another Intel SGX instruction is accessing the page. |
| \#​PF(error                                             | code) If a page fault occurs in accessing memory operands. |
| If the memory operand is not an EPC page.               |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
