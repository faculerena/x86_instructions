#EAUG
**Add a Page to an Initialized Enclave**

| Opcode/Instruction    | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                               |
| --------------------- | ----- | ---------------------- | ------------------ | --------------------------------------------------------- |
| EAX = 0DH ENCLS[EAUG] | IR    | V/V                    | SGX2               | This leaf function adds a page to an initialized enclave. |

## Instruction Operand Encoding

| Op/En | EAX       | RBX                        | RCX                                      |
| ----- | --------- | -------------------------- | ---------------------------------------- |
| IR    | EAUG (In) | Address of a PAGEINFO (In) | Address of the destination EPC page (In) |

### Description

This leaf function zeroes a page of EPC memory, associates the EPC page with an SECS page residing in the EPC, and stores the linear address and security attributes in the EPCM. As part of the association, the security attributes are configured to prevent access to the EPC page until a corresponding invocation of the EACCEPT leaf or EACCEPTCOPY leaf confirms the addition of the new page into the enclave. This instruction can only be executed when current privilege level is 0.

RBX contains the effective address of a PAGEINFO structure while RCX contains the effective address of an EPC page. The table below provides additional information on the memory parameter of the EAUG leaf function.

## EAUG Memory Parameter Semantics

| PAGEINFO                             | PAGEINFO.SECS                          | PAGEINFO.SRCPGE | PAGEINFO.SECINFO                     | EPCPAGE                           |
| ------------------------------------ | -------------------------------------- | --------------- | ------------------------------------ | --------------------------------- |
| Read access permitted by Non Enclave | Read/Write access permitted by Enclave | Must be zero    | Read access permitted by Non Enclave | Write access permitted by Enclave |

The instruction faults if any of the following:

## EAUG Faulting Conditions

| The operands are not properly aligned.    | Unsupported security attributes are set.                              |
| ----------------------------------------- | --------------------------------------------------------------------- |
| Refers to an invalid SECS.                | Reference is made to an SECS that is locked by another thread.        |
| The EPC page is locked by another thread. | RCX does not contain an effective address of an EPC page.             |
| The EPC page is already valid.            | The specified enclave offset is outside of the enclave address space. |
| The SECS has been initialized.            |                                                                       |

### Concurrency Restrictions

| Leaf                       | Parameter       | Base Concurrency Restrictions      |                             |     |
| -------------------------- | --------------- | ---------------------------------- | --------------------------- | --- |
| Access                     | On Conflict     | SGX_CONFLICT VM Exit Qualification |
| EAUG                       | Target [DS:RCX] | Exclusive \#​​​​GP                 | EPC_PAGE_CONFLICT_EXCEPTION |
| SECS [DS:RBX]PAGEINFO.SECS | Shared \#​​​​GP |                                    |

Table 38-10. Base Concurrency Restrictions of EAUG

| Leaf                                            | Parameter       | Additional Concurrency Restrictions |             |                     |             |            |     |
| ----------------------------------------------- | --------------- | ----------------------------------- | ----------- | ------------------- | ----------- | ---------- | --- |
| vs. EACCEPT, EACCEPTCOPY, EMODPE, EMODPR, EMODT |                 | vs. EADD, EEXTEND, EINIT            |             | vs. ETRACK, ETRACKC |             |
| Access                                          | On Conflict     | Access                              | On Conflict | Access              | On Conflict |
| EAUG                                            | Target [DS:RCX] | Concurrent                          |             | Concurrent          |             | Concurrent |     |
| SECS [DS:RBX]PAGEINFO.SECS                      | Concurrent      |                                     | Concurrent  |                     | Concurrent  |            |

Table 38-11. Additional Concurrency Restrictions of EAUG

### Operation

## Temp Variables in EAUG Operational Flow

| Name            | Type              | Size (bits) | Description                                                                                           |
| --------------- | ----------------- | ----------- | ----------------------------------------------------------------------------------------------------- |
| TMP_SECS        | Effective Address | 32/64       | Effective address of the SECS destination page.                                                       |
| TMP_SECINFO     | Effective Address | 32/64       | Effective address of an SECINFO structure which contains security attributes of the page to be added. |
| SCRATCH_SECINFO | SECINFO           | 512         | Scratch storage for holding the contents of DS:TMP_SECINFO.                                           |
| TMP_LINADDR     | Unsigned Integer  | 64          | Holds the linear address to be stored in the EPCM and used to calculate TMP_ENCLAVEOFFSET.            |

IF (DS:RBX is not 32Byte Aligned)

THEN #​​​​GP(0); FI;

IF (DS:RCX is not 4KByte Aligned)

THEN #​​​​GP(0); FI;

IF (DS:RCX does not resolve within an EPC)

THEN #​PF(DS:RCX); FI;

TMP_SECS := DS:RBX.SECS;

TMP_SECINFO := DS:RBX.SECINFO;

IF (DS:RBX.SECINFO is not 0)

THEN

IF (DS:TMP_SECINFO is not 64B aligned)

THEN #​​​​GP(0); FI;

FI;

TMP_LINADDR := DS:RBX.LINADDR;

IF ( DS:TMP_SECS is not 4KByte aligned or TMP_LINADDR is not 4KByte aligned )

THEN #​​​​GP(0); FI;

IF DS:RBX.SRCPAGE is not 0

THEN #​​​​GP(0); FI;

IF (DS:TMP_SECS does not resolve within an EPC)

THEN #​PF(DS:TMP_SECS); FI;

(\* Check the EPC page for concurrency \*)

IF (EPC page in use)

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

FI:

IF (EPCM(DS:RCX).VALID ≠ 0)

THEN #​PF(DS:RCX); FI;

(\* copy SECINFO contents into a scratch SECINFO \*)

IF (DS:RBX.SECINFO is 0)

THEN

(\* allocate and initialize a new scratch SECINFO structure \*)

SCRATCH_SECINFO.PT := PT_REG;

SCRATCH_SECINFO.R := 1;

SCRATCH_SECINFO.W := 1;

SCRATCH_SECINFO.X := 0;

<< zero out remaining fields of SCRATCH_SECINFO >>

ELSE

(\* copy SECINFO contents into scratch SECINFO \*)

SCRATCH_SECINFO := DS:TMP_SECINFO;

(\* check SECINFO flags for misconfiguration \*)

(\* reserved flags must be zero \*)

(\* SECINFO.FLAGS.PT must either be PT_SS_FIRST, or PT_SS_REST \*)

IF ( (SCRATCH_SECINFO reserved fields are not 0) or

CPUID.(EAX=12H, ECX=1):EAX[6] is 0) OR

(SCRATCH_SECINFO.PT is not PT_SS_FIRST, or PT_SS_REST) OR

( (SCRATCH_SECINFO.FLAGS.R is 0) OR (SCRATCH_SECINFO.FLAGS.W is 0) OR (SCRATCH_SECINFO.FLAGS.X is 1) ) )

THEN #​​​​GP(0); FI;

FI;

(\* Check if PT_SS_FIRST/PT_SS_REST page types are requested then CR4.CET must be 1 \*)

IF ( (SCRATCH_SECINFO.PT is PT_SS_FIRST OR SCRATCH_SECINFO.PT is PT_SS_REST) AND CR4.CET == 0 )

THEN #​​​​GP(0); FI;

(\* Check the SECS for concurrency \*)

IF (SECS is not available for EAUG)

THEN #​​​​GP(0); FI;

IF (EPCM(DS:TMP_SECS).VALID = 0 or EPCM(DS:TMP_SECS).PT ≠ PT_SECS)

THEN #​PF(DS:TMP_SECS); FI;

(\* Check if the enclave to which the page will be added is in the Initialized state \*)

IF (DS:TMP_SECS is not initialized)

THEN #​​​​GP(0); FI;

(\* Check the enclave offset is within the enclave linear address space \*) IF ( (TMP_LINADDR < DS:TMP_SECS.BASEADDR) or (TMP_LINADDR ≥ DS:TMP_SECS.BASEADDR + DS:TMP_SECS.SIZE) ) THEN #​​​​GP(0); FI;

IF ( (SCRATCH_SECINFO.PT is PT_SS_FIRST OR SCRATCH_SECINFO.PT is PT_SS_REST) )

THEN

(\* SS pages cannot created on first or last page of ELRANGE \*)

IF ( TMP_LINADDR == DS:TMP_SECS.BASEADDR OR

TMP_LINADDR == (DS:TMP_SECS.BASEADDR + DS:TMP_SECS.SIZE - 0x1000) )

THEN

#​​​​GP(0); FI;

FI;

(\* Clear the content of EPC page\*)

DS:RCX[32767:0] := 0;

IF (CPUID.(EAX=07H, ECX=0H):ECX[CET\_SS] = 1)

THEN

(\* set up shadow stack RSTORSSP token \*)

IF (SCRATCH_SECINFO.PT is PT_SS_FIRST)

THEN

DS:RCX[0xFF8] := (TMP_LINADDR + 0x1000) | TMP_SECS.ATTRIBUTES.MODE64BIT; FI;

FI;

(\* Set EPCM security attributes \*)

EPCM(DS:RCX).R := SCRATCH_SECINFO.FLAGS.R;

EPCM(DS:RCX).W := SCRATCH_SECINFO.FLAGS.W;

EPCM(DS:RCX).X := SCRATCH_SECINFO.FLAGS.X;

EPCM(DS:RCX).PT := SCRATCH_SECINFO.FLAGS.PT;

EPCM(DS:RCX).ENCLAVEADDRESS := TMP_LINADDR;

EPCM(DS:RCX).BLOCKED := 0;

EPCM(DS:RCX).PENDING := 1;

EPCM(DS:RCX).MODIFIED := 0;

EPCM(DS:RCX).PR := 0;

(\* associate the EPCPAGE with the SECS by storing the SECS identifier of DS:TMP_SECS \*)

Update EPCM(DS:RCX) SECS identifier to reference DS:TMP_SECS identifier;

(\* Set EPCM valid fields \*)

EPCM(DS:RCX).VALID := 1;

### Flags Affected

None

### Protected Mode Exceptions

| \#​​​​GP(0)                                  | If a memory operand effective address is outside the DS segment limit. |
| -------------------------------------------- | ---------------------------------------------------------------------- |
| If a memory operand is not properly aligned. |
| If a memory operand is locked.               |
| If the enclave is not initialized.           |
| \#​PF(error                                  | code) If a page fault occurs in accessing memory operands.             |

### 64-Bit Mode Exceptions

| \#​​​​GP(0)                                  | If a memory operand is non-canonical form.                 |
| -------------------------------------------- | ---------------------------------------------------------- |
| If a memory operand is not properly aligned. |
| If a memory operand is locked.               |
| If the enclave is not initialized.           |
| \#​PF(error                                  | code) If a page fault occurs in accessing memory operands. |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
