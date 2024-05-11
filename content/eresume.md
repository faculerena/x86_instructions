# ERESUME

**Re**

| Opcode/Instruction       | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                           |
| ------------------------ | ----- | ---------------------- | ------------------ | --------------------------------------------------------------------- |
| EAX = 03H ENCLU[ERESUME] | IR    | V/V                    | SGX1               | This leaf function is used to re-enter an enclave after an interrupt. |

## Instruction Operand Encoding

| Op/En | RAX          | RBX                   | RCX                 |
| ----- | ------------ | --------------------- | ------------------- |
| IR    | ERESUME (In) | Address of a TCS (In) | Address of AEP (In) |

### Description

The ENCLU[ERESUME] instruction resumes execution of an enclave that was interrupted due to an exception or interrupt, using the machine state previously stored in the SSA.

## ERESUME Memory Parameter Semantics

| TCS                       |
| ------------------------- |
| Enclave read/write access |

The instruction faults if any of the following occurs:

| Address in RBX is not properly aligned.                    | Any TCS.FLAGS’s must-be-zero bit is not zero.                                                   |
| ---------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| TCS pointed to by RBX is not valid or available or locked. | Current 32/64 mode does not match the enclave mode in SECS.ATTRIBUTES.MODE64.                   |
| The SECS is in use by another enclave.                     | Either of TCS-specified FS and GS segment is not a subset of the current DS segment.            |
| Any one of DS, ES, CS, SS is not zero.                     | If XSAVE available, CR4.OSXSAVE = 0, but SECS.ATTRIBUTES.XFRM ≠ 3.                              |
| CR4.OSFXSR ≠ 1.                                            | If CR4.OSXSAVE = 1, SECS.ATTRIBUTES.XFRM is not a subset of XCR0.                               |
| Offsets 520-535 of the XSAVE area not 0.                   | The bit vector stored at offset 512 of the XSAVE area must be a subset of SECS.ATTRIBUTES.XFRM. |
| The SSA frame is not valid or in use.                      | If SECS.ATTRIBUTES.AEXNOTIFY ≠ TCS.FLAGS.AEXNOTIFY and TCS.FLAGS.DBGOPTIN = 0.                  |

The following operations are performed by ERESUME:

- RSP and RBP are saved in the current SSA frame on EENTER and are automatically restored on EEXIT or an asynchronous exit due to any Interrupt event.
- The AEP contained in RCX is stored into the TCS for use by AEXs.FS and GS (including hidden portions) are saved and new values are constructed using TCS.OFSBASE/GSBASE (32 and 64-bit mode) and TCS.OFSLIMIT/GSLIMIT (32-bit mode only). The resulting segments must be a subset of the DS segment.
- If CR4.OSXSAVE == 1, XCR0 is saved and replaced by SECS.ATTRIBUTES.XFRM. The effect of RFLAGS.TF depends on whether the enclave entry is opt-in or opt-out (see Section 40.1.2):
  - On opt-out entry, TF is saved and cleared (it is restored on EEXIT or AEX). Any attempt to set TF via a POPF instruction while inside the enclave clears TF (see Section 40.2.5).
  - On opt-out entry, TF is saved and cleared (it is restored on EEXIT or AEX). Any attempt to set TF via a POPF instruction while inside the enclave clears TF (see Section 40.2.5).
  - On opt-in entry, a single-step debug exception is pended on the instruction boundary immediately after EENTER (see Section 40.2.3).
  - On opt-in entry, a single-step debug exception is pended on the instruction boundary immediately after EENTER (see Section 40.2.3).
- All code breakpoints that do not overlap with ELRANGE are also suppressed. If the entry is an opt-out entry, all code and data breakpoints that overlap with the ELRANGE are suppressed.
- On opt-out entry, a number of performance monitoring counters and behaviors are modified or suppressed (see Section 40.2.3):
  - All performance monitoring activity on the current thread is suppressed except for incrementing and firing of FIXED_CTR1 and FIXED_CTR2.
  - All performance monitoring activity on the current thread is suppressed except for incrementing and firing of FIXED_CTR1 and FIXED_CTR2.
  - PEBS is suppressed.
  - PEBS is suppressed.
  - AnyThread counting on other threads is demoted to MyThread mode and IA32_PERF_GLOBAL_STATUS[60] on that thread is set.
  - AnyThread counting on other threads is demoted to MyThread mode and IA32_PERF_GLOBAL_STATUS[60] on that thread is set.
  - If the opt-out entry on a hardware thread results in suppression of any performance monitoring, then the processor sets IA32_PERF_GLOBAL_STATUS[60] and IA32_PERF_GLOBAL_STATUS[63].
  - If the opt-out entry on a hardware thread results in suppression of any performance monitoring, then the processor sets IA32_PERF_GLOBAL_STATUS[60] and IA32_PERF_GLOBAL_STATUS[63].

### Concurrency Restrictions

| Leaf                                                     | Parameter    | Base Concurrency Restrictions |     |     |
| -------------------------------------------------------- | ------------ | ----------------------------- | --- | --- |
|                                                          | On Conflict  |                               |
| ERESUME ERESUME TCS [DS:RBX] Shared ERESUME TCS [DS:RBX] | TCS [DS:RBX] |                               |     |     |

Table 38-74. Base Concurrency Restrictions of ERESUME

| Leaf                                                                                                                                                                                     | Parameter    | Additional Concurrency Restrictions                                   |     |                     |     |            |     |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------------------------------------------------------------- | --- | ------------------- | --- | ---------- | --- |
| vs. EACCEPT, EACCEPTCOPY, vs. EADD, EEXTEND, EINIT vs. ETRACK, ETRACKC Access vs. ETRACK, ETRACKC Access On Conflict Access vs. ETRACK, ETRACKC Access On Conflict EMODPE, EMODPR, EMODT |              | vs. EADD, EEXTEND, EINIT vs. EADD, EEXTEND, EINIT vs. ETRACK, ETRACKC |     | vs. ETRACK, ETRACKC |     |
| Access On Conflict Access On Conflict Access Access On Conflict Access On Conflict                                                                                                       |              |                                                                       |     |                     |     |
| ERESUME                                                                                                                                                                                  | TCS [DS:RBX] | Concurrent                                                            |     | Concurrent          |     | Concurrent |     |

Table 38-75. Additional Concurrency Restrictions of ERESUME

### Operation

## Temp Variables in ERESUME Operational Flow

| Name              | Type              | Size  | Description                                                                  |
| ----------------- | ----------------- | ----- | ---------------------------------------------------------------------------- |
| TMP_FSBASE        | Effective Address | 32/64 | Proposed base address for FS segment.                                        |
| TMP_GSBASE        | Effective Address | 32/64 | Proposed base address for FS segment.                                        |
| TMP_FSLIMIT       | Effective Address | 32/64 | Highest legal address in proposed FS segment.                                |
| TMP_GSLIMIT       | Effective Address | 32/64 | Highest legal address in proposed GS segment.                                |
| TMP_TARGET        | Effective Address | 32/64 | Address of first instruction inside enclave at which execution is to resume. |
| TMP_SECS          | Effective Address | 32/64 | Physical address of SECS for this enclave.                                   |
| TMP_SSA           | Effective Address | 32/64 | Address of current SSA frame.                                                |
| TMP_XSIZE         | integer           | 64    | Size of XSAVE area based on SECS.ATTRIBUTES.XFRM.                            |
| TMP_SSA_PAGE      | Effective Address | 32/64 | Pointer used to iterate over the SSA pages in the current frame.             |
| TMP_GPR           | Effective Address | 32/64 | Address of the GPR area within the current SSA frame.                        |
| TMP_BRANCH_RECORD | LBR Record        |       | From/to addresses to be pushed onto the LBR stack.                           |
| TMP_NOTIFY        | Boolean           | 1     | When set to 1, deliver an AEX notification.                                  |

TMP_MODE64 := ((IA32_EFER.LMA = 1) && (CS.L = 1));

(\* Make sure DS is usable, expand up \*)

IF (TMP_MODE64 = 0 and (DS not usable or ( ( DS[S] = 1) and (DS[bit 11] = 0) and DS[bit 10] = 1))))

THEN #​​​​GP(0); FI;

(\* Check that CS, SS, DS, ES.base is 0 \*)

IF (TMP_MODE64 = 0)

THEN

IF(CS.base ≠ 0 or DS.base ≠ 0) #​​​​GP(0); FI;

IF(ES usable and ES.base ≠ 0) #​​​​GP(0); FI;

IF(SS usable and SS.base ≠ 0) #​​​​GP(0); FI;

IF(SS usable and SS.B = 0) #​​​​GP(0); FI;

FI;

IF (DS:RBX is not 4KByte Aligned)

THEN #​​​​GP(0); FI;

IF (DS:RBX does not resolve within an EPC)

THEN #​PF(DS:RBX); FI;

(\* Check AEP is canonical\*)

IF (TMP_MODE64 = 1 and (CS:RCX is not canonical))

THEN #​​​​GP(0); FI;

(\* Check concurrency of TCS operation\*)

IF (Other Intel SGX instructions are operating on TCS)

THEN #​​​​GP(0); FI;

(\* TCS verification \*)

IF (EPCM(DS:RBX).VALID = 0)

THEN #​PF(DS:RBX); FI;

IF (EPCM(DS:RBX).BLOCKED = 1)

THEN #​PF(DS:RBX); FI;

IF ((EPCM(DS:RBX).PENDING = 1) or (EPCM(DS:RBX).MODIFIED = 1))

THEN #​PF(DS:RBX); FI;

IF ( (EPCM(DS:RBX).ENCLAVEADDRESS ≠ DS:RBX) or (EPCM(DS:RBX).PT ≠ PT_TCS))

THEN #​PF(DS:RBX); FI;

IF ( (DS:RBX).OSSA is not 4KByte Aligned)

THEN #​​​​GP(0); FI;

(\* Check proposed FS and GS \*)

IF ( ( (DS:RBX).OFSBASE is not 4KByte Aligned) or ( (DS:RBX).OGSBASE is not 4KByte Aligned))

THEN #​​​​GP(0); FI;

(\* Get the SECS for the enclave in which the TCS resides \*)

TMP_SECS := Address of SECS for TCS;

(\* Make sure that the FLAGS field in the TCS does not have any reserved bits set \*)

IF ( ( (DS:RBX).FLAGS & FFFFFFFFFFFFFFFCH) ≠ 0)

THEN #​​​​GP(0); FI;

(\* SECS must exist and enclave must have previously been EINITted \*)

IF (the enclave is not already initialized)

THEN #​​​​GP(0); FI;

(\* make sure the logical processor's operating mode matches the enclave \*)

IF ( (TMP_MODE64 ≠ TMP_SECS.ATTRIBUTES.MODE64BIT))

THEN #​​​​GP(0); FI;

IF (CR4.OSFXSR = 0)

THEN #​​​​GP(0); FI;

(\* Check for legal values of SECS.ATTRIBUTES.XFRM \*)

IF (CR4.OSXSAVE = 0)

THEN

IF (TMP_SECS.ATTRIBUTES.XFRM ≠ 03H) THEN #​​​​GP(0); FI;

ELSE

IF ( (TMP_SECS.ATTRIBUTES.XFRM & XCR0) ≠ TMP_SECS.ATTRIBUTES.XFRM) THEN #​​​​GP(0); FI;

FI;

IF ( (DS:RBX).CSSA.FLAGS.DBGOPTIN = 0) and (DS:RBX).CSSA.FLAGS.AEXNOTIFY ≠ TMP_SECS.ATTRIBUTES.AEXNOTIFY))

THEN #​​​​GP(0); FI;

(\* Make sure the SSA contains at least one active frame \*)

IF ( (DS:RBX).CSSA = 0)

THEN #​​​​GP(0); FI;

(\* Compute linear address of SSA frame \*)

TMP_SSA := (DS:RBX).OSSA + TMP_SECS.BASEADDR + 4096 \* TMP_SECS.SSAFRAMESIZE \* ( (DS:RBX).CSSA - 1);

TMP_XSIZE := compute_XSAVE_frame_size(TMP_SECS.ATTRIBUTES.XFRM);

FOR EACH TMP_SSA_PAGE = TMP_SSA to TMP_SSA + TMP_XSIZE

(\* Check page is read/write accessible \*)

Check that DS:TMP_SSA_PAGE is read/write accessible;

If a fault occurs, release locks, abort and deliver that fault;

IF (DS:TMP_SSA_PAGE does not resolve to EPC page)

THEN #​PF(DS:TMP_SSA_PAGE); FI;

IF (EPCM(DS:TMP_SSA_PAGE).VALID = 0)

THEN #​PF(DS:TMP_SSA_PAGE); FI;

IF (EPCM(DS:TMP_SSA_PAGE).BLOCKED = 1)

THEN #​PF(DS:TMP_SSA_PAGE); FI;

IF ((EPCM(DS:TMP_SSA_PAGE).PENDING = 1) or (EPCM(DS:TMP_SSA_PAGE\_.MODIFIED = 1))

THEN #​PF(DS:TMP_SSA_PAGE); FI;

IF ( ( EPCM(DS:TMP_SSA_PAGE).ENCLAVEADDRESS ≠ DS:TMPSSA_PAGE) or (EPCM(DS:TMP_SSA_PAGE).PT ≠ PT_REG) or

(EPCM(DS:TMP_SSA_PAGE).ENCLAVESECS ≠ EPCM(DS:RBX).ENCLAVESECS) or

(EPCM(DS:TMP_SSA_PAGE).R = 0) or (EPCM(DS:TMP_SSA_PAGE).W = 0) )

THEN #​PF(DS:TMP_SSA_PAGE); FI;

CR_XSAVE_PAGE_n := Physical_Address(DS:TMP_SSA_PAGE);

ENDFOR

(\* Compute address of GPR area\*)

TMP_GPR := TMP_SSA + 4096 \* DS:TMP_SECS.SSAFRAMESIZE - sizeof(GPRSGX_AREA);

Check that DS:TMP_SSA_PAGE is read/write accessible;

If a fault occurs, release locks, abort and deliver that fault;

IF (DS:TMP_GPR does not resolve to EPC page)

THEN #​PF(DS:TMP_GPR); FI;

IF (EPCM(DS:TMP_GPR).VALID = 0)

THEN #​PF(DS:TMP_GPR); FI;

IF (EPCM(DS:TMP_GPR).BLOCKED = 1)

THEN #​PF(DS:TMP_GPR); FI;

IF ((EPCM(DS:TMP_GPR).PENDING = 1) or (EPCM(DS:TMP_GPR).MODIFIED = 1))

THEN #​PF(DS:TMP_GPR); FI;

IF ( ( EPCM(DS:TMP_GPR).ENCLAVEADDRESS ≠ DS:TMP_GPR) or (EPCM(DS:TMP_GPR).PT ≠ PT_REG) or

(EPCM(DS:TMP_GPR).ENCLAVESECS ≠ EPCM(DS:RBX).ENCLAVESECS) or

(EPCM(DS:TMP_GPR).R = 0) or (EPCM(DS:TMP_GPR).W = 0))

THEN #​PF(DS:TMP_GPR); FI;

IF (TMP_MODE64 = 0)

THEN

IF (TMP_GPR + (GPR_SIZE -1) is not in DS segment) THEN #​​​​GP(0); FI;

FI;

CR_GPR_PA := Physical_Address (DS: TMP_GPR);

IF ((DS:RBX).FLAGS.AEXNOTIFY = 1) and (DS:TMP_GPR.AEXNOTIFY[0] = 1))

THEN

TMP_NOTIFY := 1;

ELSE

TMP_NOTIFY := 0;

FI;

IF (TMP_NOTIFY = 1)

THEN

(\* Make sure the SSA contains at least one more frame \*)

IF ((DS:RBX).CSSA ≥ (DS:RBX).NSSA)

THEN #​​​​GP(0); FI;

TMP_SSA := TMP_SSA + 4096 \* TMP_SECS.SSAFRAMESIZE;

TMP_XSIZE := compute_XSAVE_frame_size(TMP_SECS.ATTRIBUTES.XFRM);

FOR EACH TMP_SSA_PAGE = TMP_SSA to TMP_SSA + TMP_XSIZE

(\* Check page is read/write accessible \*)

Check that DS:TMP_SSA_PAGE is read/write accessible;

If a fault occurs, release locks, abort and deliver that fault;

IF (DS:TMP_SSA_PAGE does not resolve to EPC page)

THEN #​PF(DS:TMP_SSA_PAGE); FI;

IF (EPCM(DS:TMP_SSA_PAGE).VALID = 0)

THEN #​PF(DS:TMP_SSA_PAGE); FI;

IF (EPCM(DS:TMP_SSA_PAGE).BLOCKED = 1)

THEN #​PF(DS:TMP_SSA_PAGE); FI;

IF ((EPCM(DS:TMP_SSA_PAGE).PENDING = 1) or

(EPCM(DS:TMP_SSA_PAGE).MODIFIED = 1))

THEN #​PF(DS:TMP_SSA_PAGE); FI;

IF ((EPCM(DS:TMP_SSA_PAGE).ENCLAVEADDRESS ≠ DS:TMP_SSA_PAGE) or

(EPCM(DS:TMP_SSA_PAGE).PT ≠ PT_REG) or

(EPCM(DS:TMP_SSA_PAGE).ENCLAVESECS ≠ EPCM(DS:RBX).ENCLAVESECS) or

(EPCM(DS:TMP_SSA_PAGE).R = 0) or (EPCM(DS:TMP_SSA_PAGE).W = 0))

THEN #​PF(DS:TMP_SSA_PAGE); FI;

CR_XSAVE_PAGE_n := Physical_Address(DS:TMP_SSA_PAGE);

ENDFOR

(\* Compute address of GPR area\*)

TMP_GPR := TMP_SSA + 4096 \* DS:TMP_SECS.SSAFRAMESIZE - sizeof(GPRSGX_AREA);

If a fault occurs; release locks, abort and deliver that fault;

IF (DS:TMP_GPR does not resolve to EPC page)

THEN #​PF(DS:TMP_GPR); FI;

IF (EPCM(DS:TMP_GPR).VALID = 0)

THEN #​PF(DS:TMP_GPR); FI;

IF (EPCM(DS:TMP_GPR).BLOCKED = 1)

THEN #​PF(DS:TMP_GPR); FI;

IF ((EPCM(DS:TMP_GPR).PENDING = 1) or (EPCM(DS:TMP_GPR).MODIFIED = 1))

THEN #​PF(DS:TMP_GPR); FI;

IF ((EPCM(DS:TMP_GPR).ENCLAVEADDRESS ≠ DS:TMP_GPR) or

(EPCM(DS:TMP_GPR).PT ≠ PT_REG) or

(EPCM(DS:TMP_GPR).ENCLAVESECS EPCM(DS:RBX).ENCLAVESECS) or

(EPCM(DS:TMP_GPR).R = 0) or (EPCM(DS:TMP_GPR).W = 0))

THEN #​PF(DS:TMP_GPR); FI;

IF (TMP_MODE64 = 0)

THEN

IF (TMP_GPR + (GPR_SIZE -1) is not in DS segment) THEN #​​​​GP(0); FI;

FI;

CR_GPR_PA := Physical_Address (DS: TMP_GPR);

TMP_TARGET := (DS:RBX).OENTRY + TMP_SECS.BASEADDR;

ELSE

TMP_TARGET := (DS:TMP_GPR).RIP;

FI;

IF (TMP_MODE64 = 1)

THEN

IF (TMP_TARGET is not canonical) THEN #​​​​GP(0); FI;

ELSE

IF (TMP_TARGET > CS limit) THEN #​​​​GP(0); FI;

FI;

(\* Check proposed FS/GS segments fall within DS \*)

IF (TMP_MODE64 = 0)

THEN

TMP_FSBASE := (DS:RBX).OFSBASE + TMP_SECS.BASEADDR;

TMP_FSLIMIT := (DS:RBX).OFSBASE + TMP_SECS.BASEADDR + (DS:RBX).FSLIMIT;

TMP_GSBASE := (DS:RBX).OGSBASE + TMP_SECS.BASEADDR;

TMP_GSLIMIT := (DS:RBX).OGSBASE + TMP_SECS.BASEADDR + (DS:RBX).GSLIMIT;

(\* if FS wrap-around, make sure DS has no holes\*)

IF (TMP_FSLIMIT < TMP_FSBASE)

THEN

IF (DS.limit < 4GB) THEN #​​​​GP(0); FI;

ELSE

IF (TMP_FSLIMIT > DS.limit) THEN #​​​​GP(0); FI;

FI;

(\* if GS wrap-around, make sure DS has no holes\*)

IF (TMP_GSLIMIT < TMP_GSBASE)

THEN

IF (DS.limit < 4GB) THEN #​​​​GP(0); FI;

ELSE

IF (TMP_GSLIMIT > DS.limit) THEN #​​​​GP(0); FI;

FI;

ELSE

IF (TMP_NOTIFY = 1)

THEN

TMP_FSBASE := (DS:RBX).OFSBASE + TMP_SECS.BASEADDR;

TMP_GSBASE := (DS:RBX).OGSBASE + TMP_SECS.BASEADDR;

ELSE

TMP_FSBASE := DS:TMP_GPR.FSBASE;

TMP_GSBASE := DS:TMP_GPR.GSBASE;

FI;

IF ((TMP_FSBASE is not canonical) or (TMP_GSBASE is not canonical))

THEN #​​​​GP(0); FI;

FI;

(\* Ensure the enclave is not already active and this thread is the only one using the TCS\*)

IF (DS:RBX.STATE = ACTIVE))

THEN #​​​​GP(0); FI;

TMP_IA32_U_CET := 0

TMP_SSP := 0

IF (CPUID.(EAX=12H, ECX=1):EAX[6] = 1)

THEN

IF ( CR4.CET = 0 )

THEN

(\* If part does not support CET or CET has not been enabled and enclave requires CET then fail \*)

IF (TMP_SECS.CET_ATTRIBUTES ≠ 0 OR TMP_SECS.CET_LEG_BITMAP_OFFSET ≠ 0) #​​​​GP(0); FI;

FI;

(\* If indirect branch tracking or shadow stacks enabled but CET state save area is not 16B aligned then fail ERESUME \*)

IF (TMP_SECS.CET_ATTRIBUTES.SH_STK_EN = 1 OR TMP_SECS.CET_ATTRIBUTES.ENDBR_EN = 1)

THEN

IF (DS:RBX.OCETSSA is not 16B aligned) #​​​​GP(0); FI;

FI;

IF (TMP_SECS.CET_ATTRIBUTES.SH_STK_EN OR TMP_SECS.CET_ATTRIBUTES.ENDBR_EN)

THEN

(\* Setup CET state from SECS, note tracker goes to IDLE \*)

TMP_IA32_U_CET = TMP_SECS.CET_ATTRIBUTES;

IF (TMP_IA32_U_CET.LEG_IW_EN = 1 AND TMP_IA32_U_CET.ENDBR_EN = 1)

THEN

TMP_IA32_U_CET := TMP_IA32_U_CET + TMP_SECS.BASEADDR;

TMP_IA32_U_CET := TMP_IA32_U_CET + TMP_SECS.CET_LEG_BITMAP_BASE;

FI;

(\* Compute linear address of what will become new CET state save area and cache its PA \*)

IF (TMP_NOTIFY = 1)

THEN

TMP_CET_SAVE_AREA = DS:RBX.OCETSSA + TMP_SECS.BASEADDR + (DS:RBX.CSSA) \* 16;

ELSE

TMP_CET_SAVE_AREA = DS:RBX.OCETSSA + TMP_SECS.BASEADDR + (DS:RBX.CSSA - 1) \* 16;

FI;

TMP_CET_SAVE_PAGE = TMP_CET_SAVE_AREA & ~0xFFF;

Check the TMP_CET_SAVE_PAGE page is read/write accessible

If fault occurs release locks, abort and deliver fault

(\* read the EPCM VALID, PENDING, MODIFIED, BLOCKED and PT fields atomically \*)

IF ((DS:TMP_CET_SAVE_PAGE Does NOT RESOLVE TO EPC PAGE) OR

(EPCM(DS:TMP_CET_SAVE_PAGE).VALID = 0) OR

(EPCM(DS:TMP_CET_SAVE_PAGE).PENDING = 1) OR

(EPCM(DS:TMP_CET_SAVE_PAGE).MODIFIED = 1) OR

(EPCM(DS:TMP_CET_SAVE_PAGE).BLOCKED = 1) OR

(EPCM(DS:TMP_CET_SAVE_PAGE).R = 0) OR

(EPCM(DS:TMP_CET_SAVE_PAGE).W = 0) OR

(EPCM(DS:TMP_CET_SAVE_PAGE).ENCLAVEADDRESS ≠ DS:TMP_CET_SAVE_PAGE) OR

(EPCM(DS:TMP_CET_SAVE_PAGE).PT ≠ PT_SS_REST) OR

(EPCM(DS:TMP_CET_SAVE_PAGE).ENCLAVESECS ≠ EPCM(DS:RBX).ENCLAVESECS))

THEN

#​PF(DS:TMP_CET_SAVE_PAGE);

FI;

CR_CET_SAVE_AREA_PA := Physical address(DS:TMP_CET_SAVE_AREA)

IF (TMP_NOTIFY = 1)

THEN

IF TMP_IA32_U_CET.SH_STK_EN = 1

THEN TMP_SSP = TCS.PREVSSP; FI;

ELSE

TMP_SSP = CR_CET_SAVE_AREA_PA.SSP

TMP_IA32_U_CET.TRACKER = CR_CET_SAVE_AREA_PA.TRACKER;

TMP_IA32_U_CET.SUPPRESS = CR_CET_SAVE_AREA_PA.SUPPRESS;

IF ( (TMP_MODE64 = 1 AND TMP_SSP is not canonical) OR

(TMP_MODE64 = 0 AND (TMP_SSP & 0xFFFFFFFF00000000) ≠ 0) OR

(TMP_SSP is not 4 byte aligned) OR

(TMP_IA32_U_CET.TRACKER = WAIT_FOR_ENDBRANCH AND TMP_IA32_U_CET.SUPPRESS = 1) OR

(CR_CET_SAVE_AREA_PA.Reserved ≠ 0) ) #​​​​GP(0); FI;

FI;

FI;

FI;

IF (TMP_NOTIFY = 0)

THEN

(\* SECS.ATTRIBUTES.XFRM selects the features to be saved. \*)

(\* CR_XSAVE_PAGE_n: A list of 1 or more physical address of pages that contain the XSAVE area. \*)

XRSTOR(TMP_MODE64, SECS.ATTRIBUTES.XFRM, CR_XSAVE_PAGE_n);

IF (XRSTOR failed with #​​​​GP)

THEN

DS:RBX.STATE := INACTIVE;

#​​​​GP(0);

FI;

FI;

CR_ENCLAVE_MODE := 1;

CR_ACTIVE_SECS := TMP_SECS;

CR_ELRANGE := (TMP_SECS.BASEADDR, TMP_SECS.SIZE);

(\* Save sate for possible AEXs \*)

CR_TCS_PA := Physical_Address (DS:RBX);

CR_TCS_LA := RBX;

CR_TCS_LA.AEP := RCX;

(\* Save the hidden portions of FS and GS \*)

CR_SAVE_FS_selector := FS.selector;

CR_SAVE_FS_base := FS.base;

CR_SAVE_FS_limit := FS.limit;

CR_SAVE_FS_access_rights := FS.access_rights;

CR_SAVE_GS_selector := GS.selector;

CR_SAVE_GS_base := GS.base;

CR_SAVE_GS_limit := GS.limit;

CR_SAVE_GS_access_rights := GS.access_rights;

IF (TMP_NOTIFY = 1)

THEN

(\* If XSAVE is enabled, save XCR0 and replace it with SECS.ATTRIBUTES.XFRM\*)

IF (CR4.OSXSAVE = 1)

THEN

CR_SAVE_XCR0 := XCR0;

XCR0 := TMP_SECS.ATTRIBUTES.XFRM;

FI;

FI;

RIP := TMP_TARGET;

IF (TMP_NOTIFY = 1)

THEN

RCX := RIP;

RAX := (DS:RBX).CSSA;

(\* Save the outside RSP and RBP so they can be restored on interrupt or EEXIT \*)

DS:TMP_SSA.U_RSP := RSP;

DS:TMP_SSA.U_RBP := RBP;

ELSE

Restore_GPRs from DS:TMP_GPR;

(\*Restore the RFLAGS values from SSA\*)

RFLAGS.CF := DS:TMP_GPR.RFLAGS.CF;

RFLAGS.PF := DS:TMP_GPR.RFLAGS.PF;

RFLAGS.AF := DS:TMP_GPR.RFLAGS.AF;

RFLAGS.ZF := DS:TMP_GPR.RFLAGS.ZF;

RFLAGS.SF := DS:TMP_GPR.RFLAGS.SF;

RFLAGS.DF := DS:TMP_GPR.RFLAGS.DF;

RFLAGS.OF := DS:TMP_GPR.RFLAGS.OF;

RFLAGS.NT := DS:TMP_GPR.RFLAGS.NT;

RFLAGS.AC := DS:TMP_GPR.RFLAGS.AC;

RFLAGS.ID := DS:TMP_GPR.RFLAGS.ID;

RFLAGS.RF := DS:TMP_GPR.RFLAGS.RF;

RFLAGS.VM := 0;

IF (RFLAGS.IOPL = 3)

THEN RFLAGS.IF := DS:TMP_GPR.RFLAGS.IF; FI;

IF (TCS.FLAGS.OPTIN = 0)

THEN RFLAGS.TF := 0; FI;

(\* If XSAVE is enabled, save XCR0 and replace it with SECS.ATTRIBUTES.XFRM\*)

IF (CR4.OSXSAVE = 1)

THEN

CR_SAVE_XCR0 := XCR0;

XCR0 := TMP_SECS.ATTRIBUTES.XFRM;

FI;

(\* Pop the SSA stack\*)

(DS:RBX).CSSA := (DS:RBX).CSSA -1;

FI;

(\* Do the FS/GS swap \*)

FS.base := TMP_FSBASE;

FS.limit := DS:RBX.FSLIMIT;

FS.type := 0001b;

FS.W := DS.W;

FS.S := 1;

FS.DPL := DS.DPL;

FS.G := 1;

FS.B := 1;

FS.P := 1;

FS.AVL := DS.AVL;

FS.L := DS.L;

FS.unusable := 0;

FS.selector := 0BH;

GS.base := TMP_GSBASE;

GS.limit := DS:RBX.GSLIMIT;

GS.type := 0001b;

GS.W := DS.W;

GS.S := 1;

GS.DPL := DS.DPL;

GS.G := 1;

GS.B := 1;

GS.P := 1;

GS.AVL := DS.AVL;

GS.L := DS.L;

GS.unusable := 0;

GS.selector := 0BH;

CR_DBGOPTIN := TCS.FLAGS.DBGOPTIN;

Suppress all code breakpoints that are outside ELRANGE;

IF (CR_DBGOPTIN = 0)

THEN

Suppress all code breakpoints that overlap with ELRANGE;

CR_SAVE_TF := RFLAGS.TF;

RFLAGS.TF := 0;

Suppress any MTF VM exits during execution of the enclave;

Clear all pending debug exceptions;

Clear any pending MTF VM exit;

ELSE

IF (TMP_NOTIFY = 1)

THEN

IF RFLAGS.TF = 1

THEN pend a single-step #​DB at the end of ERESUME; FI;

IF the “monitor trap flag” VM-execution control is set

THEN pend an MTF VM exit at the end of ERESUME; FI;

ELSE

Clear all pending debug exceptions;

Clear pending MTF VM exits;

FI;

FI;

IF ((CPUID.(EAX=7H, ECX=0):EDX[CET\_IBT] = 1) OR (CPUID.(EAX=7, ECX=0):ECX[CET\_SS] = 1)

THEN

(\* Save enclosing application CET state into save registers \*)

CR_SAVE_IA32_U_CET := IA32_U_CET

(\* Setup enclave CET state \*)

IF CPUID.(EAX=07H, ECX=00h):ECX[CET\_SS] = 1

THEN

CR_SAVE_SSP := SSP

SSP := TMP_SSP;

FI;

IA32_U_CET := TMP_IA32_U_CET;

FI;

(\* Assure consistent translations \*)

Flush_linear_context;

Clear_Monitor_FSM;

Allow_front_end_to_begin_fetch_at_new_RIP;

### Flags Affected

RFLAGS.TF is cleared on opt-out entry

### Protected Mode Exceptions

| \#​​​​GP(0)                                                                                                            | If DS:RBX is not page aligned.                    |
| ---------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| If the enclave is not initialized.                                                                                     |
| If the thread is not in the INACTIVE state.                                                                            |
| If CS, DS, ES or SS bases are not all zero.                                                                            |
| If executed in enclave mode.                                                                                           |
| If part or all of the FS or GS segment specified by TCS is outside the DS segment.                                     |
| If any reserved field in the TCS FLAG is set.                                                                          |
| If the target address is not within the CS segment.                                                                    |
| If CR4.OSFXSR = 0.                                                                                                     |
| If CR4.OSXSAVE = 0 and SECS.ATTRIBUTES.XFRM ≠ 3.                                                                       |
| If CR4.OSXSAVE = 1and SECS.ATTRIBUTES.XFRM is not a subset of XCR0.                                                    |
| If SECS.ATTRIBUTES.AEXNOTIFY ≠ TCS.FLAGS.AEXNOTIFY and TCS.FLAGS.DBGOPTIN = 0.                                         |
| \#​PF(error                                                                                                            | code) If a page fault occurs in accessing memory. |
| If DS:RBX does not point to a valid TCS.                                                                               |
| If one or more pages of the current SSA frame are not readable/writable, or do not resolve to a valid PT_REG EPC page. |

### 64-Bit Mode Exceptions

| \#​​​​GP(0)                                                                                                            | If DS:RBX is not page aligned.                             |
| ---------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| If the enclave is not initialized.                                                                                     |
| If the thread is not in the INACTIVE state.                                                                            |
| If CS, DS, ES or SS bases are not all zero.                                                                            |
| If executed in enclave mode.                                                                                           |
| If part or all of the FS or GS segment specified by TCS is outside the DS segment.                                     |
| If any reserved field in the TCS FLAG is set.                                                                          |
| If the target address is not canonical.                                                                                |
| If CR4.OSFXSR = 0.                                                                                                     |
| If CR4.OSXSAVE = 0 and SECS.ATTRIBUTES.XFRM ≠ 3.                                                                       |
| If CR4.OSXSAVE = 1and SECS.ATTRIBUTES.XFRM is not a subset of XCR0.                                                    |
| If SECS.ATTRIBUTES.AEXNOTIFY ≠ TCS.FLAGS.AEXNOTIFY and TCS.FLAGS.DBGOPTIN = 0.                                         |
| \#​PF(error                                                                                                            | code) If a page fault occurs in accessing memory operands. |
| If DS:RBX does not point to a valid TCS.                                                                               |
| If one or more pages of the current SSA frame are not readable/writable, or do not resolve to a valid PT_REG EPC page. |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
