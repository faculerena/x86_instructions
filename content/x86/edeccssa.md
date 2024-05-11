# EDECCSSA**Decrements TCS**

| Opcode/Instruction        | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                             |
| ------------------------- | ----- | ---------------------- | ------------------ | --------------------------------------- |
| EAX = 09H ENCLU[EDECCSSA] | IR    | V/V                    | EDECCSSA           | This leaf function decrements TCS.CSSA. |

## Instruction Operand Encoding

| Op/En | EAX           |
| ----- | ------------- |
| IR    | EDECCSSA (In) |

### Description

This leaf function changes the current SSA frame by decrementing TCS.CSSA for the current enclave thread. If the enclave has enabled CET shadow stacks or indirect branch tracking, then EDECCSSA also changes the current CET state save frame. This instruction leaf can only be executed inside an enclave.

## EDECCSSA Memory Parameter Semantics

| TCS                          |
| ---------------------------- |
| Read/Write access by Enclave |

The instruction faults if any of the following:

## EDECCSSA Faulting Conditions

| TCS.CSSA is 0.                        | TCS is not valid or available or locked. |
| ------------------------------------- | ---------------------------------------- |
| The SSA frame is not valid or in use. |                                          |

### Concurrency Restrictions

| Leaf                                                                  | Parameter         | Base Concurrency Restrictions |     |     |
| --------------------------------------------------------------------- | ----------------- | ----------------------------- | --- | --- |
|                                                                       | On Conflict       |                               |
| EDECCSSA EDECCSSA TCS [CR\_TCS\_PA] Shared EDECCSSA TCS [CR\_TCS\_PA] | TCS [CR\_TCS\_PA] |                               |     |     |

Table 38-60. Base Concurrency Restrictions of EDECCSSA

| Leaf                                                                                                                                                                                     | Parameter         | Additional Concurrency Restrictions                                   |     |                     |     |            |     |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------------------------------------------------------------- | --- | ------------------- | --- | ---------- | --- |
| vs. EACCEPT, EACCEPTCOPY, vs. EADD, EEXTEND, EINIT vs. ETRACK, ETRACKC Access vs. ETRACK, ETRACKC Access On Conflict Access vs. ETRACK, ETRACKC Access On Conflict EMODPE, EMODPR, EMODT |                   | vs. EADD, EEXTEND, EINIT vs. EADD, EEXTEND, EINIT vs. ETRACK, ETRACKC |     | vs. ETRACK, ETRACKC |     |
| Access On Conflict Access On Conflict Access Access On Conflict Access On Conflict                                                                                                       |                   |                                                                       |     |                     |     |
| EDECCSSA                                                                                                                                                                                 | TCS [CR\_TCS\_PA] | Concurrent                                                            |     | Concurrent          |     | Concurrent |     |

Table 38-61. Additional Concurrency Restrictions of EDECCSSA

### Operation

## Temp Variables in EDECCSSA Operational Flow

| Name                | Type              | Size (bits) | Description                                                     |
| ------------------- | ----------------- | ----------- | --------------------------------------------------------------- |
| TMP_SSA             | Effective Address | 32/64       | Address of current SSA frame.                                   |
| TMP_XSIZE           | Integer           | 64          | Size of XSAVE area based on SECS.ATTRIBUTES.XFRM.               |
| TMP_SSA_PAGE        | Effective Address | 32/64       | Pointer used to iterate over the SSA pages in the target frame. |
| TMP_GPR             | Effective Address | 32/64       | Address of the GPR area within the target SSA frame.            |
| TMP_XSAVE_PAGE_PA_n | Physical Address  | 32/64       | Physical address of the nth page within the target SSA frame.   |
| TMP_CET_SAVE_AREA   | Effective Address | 32/64       | Address of the current CET save area.                           |
| TMP_CET_SAVE_PAGE   | Effective Address | 32/64       | Address of the current CET save area page.                      |

IF (CR_TCS_PA.CSSA = 0)

THEN #​​​​GP(0); FI;

(\* Compute linear address of SSA frame \*)

TMP_SSA := CR_TCS_PA.OSSA + CR_ACTIVE_SECS.BASEADDR + 4096 \* CR_ACTIVE_SECS.SSAFRAMESIZE \* (CR_TCS_PA.CSSA - 1);

TMP_XSIZE := compute_XSAVE_frame_size(CR_ACTIVE_SECS.ATTRIBUTES.XFRM);

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

IF ( ( EPCM(DS:TMP_SSA_PAGE).ENCLAVEADDRESS ≠ DS:TMPSSA_PAGE) or

(EPCM(DS:TMP_SSA_PAGE).PT ≠ PT_REG) or

(EPCM(DS:TMP_SSA_PAGE).ENCLAVESECS ≠ EPCM(CR_TCS_PA).ENCLAVESECS) or

(EPCM(DS:TMP_SSA_PAGE).R = 0) or (EPCM(DS:TMP_SSA_PAGE).W = 0))

THEN #​PF(DS:TMP_SSA_PAGE); FI;

TMP_XSAVE_PAGE_PA_n := Physical_Address(DS:TMP_SSA_PAGE);

ENDFOR

(\* Compute address of GPR area\*)

TMP_GPR := TMP_SSA + 4096 \* CR_ACTIVE_SECS.SSAFRAMESIZE - sizeof(GPRSGX_AREA);

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

IF ( ( EPCM(DS:TMP_GPR).ENCLAVEADDRESS ≠ DS:TMP_GPR) or

(EPCM(DS:TMP_GPR).PT ≠ PT_REG) or

(EPCM(DS:TMP_GPR).ENCLAVESECS ≠ EPCM(CR_TCS_PA).ENCLAVESECS) or

(EPCM(DS:TMP_GPR).R = 0) or (EPCM(DS:TMP_GPR).W = 0) )

THEN #​PF(DS:TMP_GPR); FI;

IF (TMP_MODE64 = 0)

THEN

IF (TMP_GPR + (sizeof(GPRSGX_AREA) -1) is not in DS segment)

THEN #​​​​GP(0); FI;

FI;

IF (CPUID.(EAX=12H, ECX=1):EAX[6] = 1)

THEN

IF ((CR_ACTIVE_SECS.CET_ATTRIBUTES.SH_STK_EN == 1) OR (CR_ACTIVE_SECS.CET_ATTRIBUTES.ENDBR_EN == 1))

THEN

(\* Compute linear address of what will become new CET state save area and cache its PA \*)

TMP_CET_SAVE_AREA := CR_TCS_PA.OCETSSA + CR_ACTIVE_SECS.BASEADDR + (CR_TCS_PA.CSSA - 1) \* 16;

TMP_CET_SAVE_PAGE := TMP_CET_SAVE_AREA & ~0xFFF;

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

(EPCM(DS:TMP_CET_SAVE_PAGE).ENCLAVESECS ≠ EPCM(CR_TCS_PA).ENCLAVESECS))

THEN #​PF(DS:TMP_CET_SAVE_PAGE); FI;

FI;

FI;

(\* At this point, the instruction is guaranteed to complete \*)

CR_TCS_PA.CSSA := CR_TCS_PA.CSSA - 1;

CR_GPR_PA := Physical_Address(DS:TMP_GPR);

FOR EACH TMP_XSAVE_PAGE_n

CR_XSAVE_PAGE_n := TMP_XSAVE_PAGE_PA_n;

ENDFOR

IF (CPUID.(EAX=12H, ECX=1):EAX[6] = 1)

IF ((TMP_SECS.CET_ATTRIBUTES.SH_STK_EN == 1) OR

(TMP_SECS.CET_ATTRIBUTES.ENDBR_EN == 1))

THEN

CR_CET_SAVE_AREA_PA := Physical_Address(DS:TMP_CET_SAVE_AREA);

FI;

FI;

### Flags Affected

None

### Protected Mode Exceptions

| \#​​​​GP(0)                                                                                                                              | If executed outside an enclave.                   |
| ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| If CR_TCS_PA.CSSA = 0.                                                                                                                   |
| \#​PF(error                                                                                                                              | code) If a page fault occurs in accessing memory. |
| If one or more pages of the target SSA frame are not readable/writable, or do not resolve to a valid PT_REG EPC page.                    |
| If CET is enabled for the enclave and the target CET SSA frame is not readable/writable, or does not resolve to a valid PT_REG EPC page. |

### 64-Bit Mode Exceptions

| \#​​​​GP(0)                                                                                                                              | If executed outside an enclave.                   |
| ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| If CR_TCS_PA.CSSA = 0.                                                                                                                   |
| \#​PF(error                                                                                                                              | code) If a page fault occurs in accessing memory. |
| If one or more pages of the target SSA frame are not readable/writable, or do not resolve to a valid PT_REG EPC page.                    |
| If CET is enabled for the enclave and the target CET SSA frame is not readable/writable, or does not resolve to a valid PT_REG EPC page. |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
