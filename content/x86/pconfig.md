# PCONFIG**Platform Configuration**

| Opcode/Instruction  | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                      |
| ------------------- | ----- | ---------------------- | ------------------ | -------------------------------------------------------------------------------- |
| NP 0F 01 C5 PCONFIG | A     | V/V                    | PCONFIG            | This instruction is used to execute functions for configuring platform features. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1 | Operand 2 | Operand 3 | Operand 4 |
| ----- | ----- | --------- | --------- | --------- | --------- |
| A     | N/A   | N/A       | N/A       | N/A       | N/A       |

## Description

The PCONFIG instruction allows software to configure certain platform features. It supports these features with multiple leaf functions, selecting a leaf function using the value in EAX.

Depending on the leaf function, the registers RBX, RCX, and RDX may be used to provide input information or for the instruction to report output information. Addresses and operands are 32 bits outside 64-bit mode and are 64 bits in 64-bit mode. The value of CS.D does not affect operand size or address size.

Executions of PCONFIG may fail for platform-specific reasons. An execution reports failure by setting the ZF flag and loading EAX with a non-zero failure reason; a successful execution clears ZF and EAX.

Each PCONFIG leaf function applies to a specific hardware block called a PCONFIG target. The leaf function is supported only if the processor supports that target. Each target is associated with a numerical target identifier, and CPUID leaf 1BH (PCONFIG information) enumerates the identifiers of the supported targets. An attempt to execute an undefined leaf function, or a leaf function that applies to an unsupported target identifier, results in a general-protection exception (#​​​​GP).

## Leaf Function MKTME_KEY_PROGRAM

As of this writing, the only defined PCONFIG leaf function is used for key programming for total memory encryption-multi-key (TME-MK).1 This leaf function is called MKTME_KEY_PROGRAM and it pertains to the TME-MK target, which has target identifier 1. The leaf function is selected by loading EAX with value 0. The MKTME_KEY_PROGRAM leaf function uses the EBX (or RBX) register for additional input information.

Software uses the MKTME_KEY_PROGRAM leaf function to manage the encryption key associated with a particular key identifier (KeyID). The leaf function uses a data structure called the **TME-MK key programming structure** (MKTME_KEY_PROGRAM_STRUCT). Software provides the address of the structure (as an offset in the DS segment) in EBX (or RBX). The format of the structure is given in [Table 4-15](/x86/pconfig#tbl-4-15).

| Field       | Offset (bytes) | Size (bytes) | Comments                                                                                                                                             |
| ----------- | -------------- | ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| KEYID       | 0              | 2            | Key Identifier.                                                                                                                                      |
| KEYID_CTRL  | 2              | 4            | KeyID control: • Bits 7:0: key-programming command (COMMAND) • Bits 23:8: encryption algorithm (ENC_ALG) • Bits 31:24: Reserved, must be zero (RSVD) |
| Ignored     | 6              | 58           | Not used.                                                                                                                                            |
| KEY_FIELD_1 | 64             | 64           | Software supplied data key or entropy for data key.                                                                                                  |
| KEY_FIELD_2 | 128            | 64           | Software supplied tweak key or entropy for tweak key.                                                                                                |

[Table 4-15](/x86/pconfig#tbl-4-15). MKTME_KEY_PROGRAM_STRUCT Format

1. Further details on TME-MK can be found here:

### https://software.intel.com/sites/default/files/managed/a5/16/Multi-Key-Total-Memory-Encryption-Spec.pdf

A description of each of the fields in MKTME_KEY_PROGRAM_STRUCT is provided below:

- **KEYID:** The key identifier (KeyID) being programmed to the MKTME engine. PCONFIG causes a general-protection exception (#​​​​GP) if the KeyID is zero. KeyID zero always uses the current behavior configured for TME (total memory encryption), either to encrypt with platform TME key or to bypass TME encryption. PCONFIG also causes a #​​​​GP if the KeyID exceeds the maximum enumerated in IA32_TME_CAPABILITY.MK_TME_MAX-\_KEYS[bits 50:36] or configured by the setting of IA32_TME_ACTIVATE.MK_TME_KEYID_BITS[bits 35:32].
- **KEYID_CTRL:** The KEYID_CTRL field comprises two sub-fields used by software to control the encryption performed for the selected KeyID:
  - Key-programming command (COMMAND; bits 7:0). This 8-bit field should contain one of the following values:
  - Key-programming command (COMMAND; bits 7:0). This 8-bit field should contain one of the following values:
- KEYID_SET_KEY_DIRECT (value 0). With this command, software programs directly the encryption key to be used for the selected KeyID.
- KEYID_SET_KEY_RANDOM (value 1). With this command, software has the CPU generate and assign an encryption key to be used for the selected KeyID using a hardware random-number generator.

If this command is used and there is insufficient entropy for the random-number generator, PCONFIG will fail and report the failure by loading EAX with value 2 (ENTROPY_ERROR).

Because the keys programed by PCONFIG are discarded on reset and software cannot read the programmed keys, the keys programmed with this command are ephemeral.

- KEYID_CLEAR_KEY (value 2). With this command, software indicates that the selected KeyID should use the current behavior configured for TME (see above).
- KEYID_NO_ENCRYPT (value 3). With this command, software indicates that no encryption should be used for the selected KeyID.

If any other value is used, PCONFIG causes a #​​​​GP.

— Encryption algorithm (ENC_ALG, bits 23:8). Bits 63:48 of the IA32_TME_ACTIVATE MSR (MSR index 982H) indicate which encryption algorithms are supported by the platform. The 16-bit ENC_ALG field should specify one of the algorithms indicated in IA32_TME_ACTIVATE. PCONFIG causes a #​​​​GP if ENC_ALG does not set exactly one bit or if it sets a bit whose corresponding bit is not set in IA32_TME_ACTIVATE[63:48].

- **KEY_FIELD_1:** Use of this field depends upon selected key-programming command:
  - If the direct key-programming command is used (KEYID_SET_KEY_DIRECT), this field carries the software supplied data key to be used for the KeyID.
  - If the direct key-programming command is used (KEYID_SET_KEY_DIRECT), this field carries the software supplied data key to be used for the KeyID.
  - If the random key-programming command is used (KEYID_SET_KEY_RANDOM), this field carries the software supplied entropy to be mixed in the CPU generated random data key.
  - If the random key-programming command is used (KEYID_SET_KEY_RANDOM), this field carries the software supplied entropy to be mixed in the CPU generated random data key.
  - This field is ignored when one of the other key-programming commands is used.
  - This field is ignored when one of the other key-programming commands is used.

It is software’s responsibility to ensure that the key supplied for the direct key-programming option or the entropy supplied for the random key-programming option does not result in weak keys. There are no explicit checks in the instruction to detect or prevent weak keys.

- **KEY_FIELD_2:** Use of this field depends upon selected key-programming command:
  - If the direct key-programming command is used (KEYID_SET_KEY_DIRECT), this field carries the software supplied tweak key to be used for the KeyID.
  - If the direct key-programming command is used (KEYID_SET_KEY_DIRECT), this field carries the software supplied tweak key to be used for the KeyID.
  - If the random key-programming command is used (KEYID_SET_KEY_RANDOM), this field carries the software supplied entropy to be mixed in the CPU generated random tweak key.
  - If the random key-programming command is used (KEYID_SET_KEY_RANDOM), this field carries the software supplied entropy to be mixed in the CPU generated random tweak key.
  - This field is ignored when one of the other key-programming commands is used.
  - This field is ignored when one of the other key-programming commands is used.

It is software’s responsibility to ensure that the key supplied for the direct key-programming option or the entropy supplied for the random key-programming option does not result in weak keys. There are no explicit checks in the instruction to detect or prevent weak keys.

All KeyIDs default to TME behavior (encrypt with TME key or bypass encryption) on activation of TME-MK. Software can at any point decide to change the key for a KeyID using the MKTME_KEY_PROGRAM leaf function of the PCONFIG instruction. Changing the key for a KeyID does **not** change the state of the TLB caches or memory pipeline. Software is responsible for taking appropriate actions to ensure correct behavior.

The key table used by TME-MK is shared by all logical processors in a platform. For this reason, execution of the MKTME_KEY_PROGRAM leaf function must gain exclusive access to the key table before updating it. The leaf function does this by acquiring lock (implemented in the platform) and retaining that lock until the execution completes. An execution of the leaf function may fail to acquire the lock if it is already in use. In this situation, the leaf function will load EAX with failure reason 5 (DEVICE_BUSY) indicating that software must retry. When this happens, the key table is not updated, and software should retry execution of PCONFIG.

> Earlier versions of this manual specified that bytes 63:6 of MKTME_KEY_PROGRAM_STRUCT were reserved and that PCONFIG would cause a #​​​​GP if they were not all zero. This is not the case. As indicated in [Table 4-15](/x86/pconfig#tbl-4-15), PCONFIG ignores those bytes.
>
> They also specified that PCONFIG would cause a #​​​​GP if the upper 48 bytes of each of the 64-byte key fields were not all 0. This is not the case. From each of these fields, PCONFIG uses the number of bytes required by the selected encryption algorithm (e.g., 32 bytes for AES-XTS 256) and ignores the upper bytes.
>
> They also specified that PCONFIG would complete and report a failure reason in EAX if the structure specified an incorrect KeyID, and unsupported key-programming command, or an incorrect selection of an encryption algorithm. This is not the case. As indicated above (and in the Operation section), those conditions cause #​​​​GP.

## Operation

```
(* #​​​UD if PCONFIG is not enumerated or CPL > 0 *)
IF CPUID.7.0:EDX[18] = 0 OR CPL > 0
    THEN #​​​UD; FI;
(* #​​​​GP(0) for an unsupported leaf function *)
IF EAX != 0
    THEN #​​​​GP(0); FI;
CASE (EAX) (* operation based on selected leaf function *)
    0 (MKTME_KEY_PROGRAM):
    (* Confirm that TME-MK is properly enabled by the IA32_TME_ACTIVATE MSR *)
    (* The MSR must be locked, encryption enabled, and a non-zero number of KeyID bits specified *)
    IF IA32_TME_ACTIVATE[0] = 0 OR IA32_TME_ACTIVATE[1] = 0 OR IA32_TME_ACTIVATE[35:32] = 0
            THEN #​​​​GP(0); FI;
    IF DS:RBX is not 256-byte aligned
        THEN #​​​​GP(0); FI;
    Load TMP_KEY_PROGRAM_STRUCT from 192 bytes at linear address DS:RBX;
    IF TMP_KEY_PROGRAM_STRUCT.KEYID_CTRL sets any reserved bits
        THEN #​​​​GP(0); FI;
    (* Check for a valid command *)
    IF TMP_KEY_PROGRAM_STRUCT. KEYID_CTRL.COMMAND > 3
        THEN #​​​​GP(0); FI;
    (* Check that the KEYID being operated upon is a valid KEYID *)
    IF TMP_KEY_PROGRAM_STRUCT.KEYID = 0 OR
        TMP_KEY_PROGRAM_STRUCT.KEYID > 2^IA32_TME_ACTIVATE.MK_TME_KEYID_BITS – 1 OR
        TMP_KEY_PROGRAM_STRUCT.KEYID > IA32_TME_CAPABILITY.MK_TME_MAX_KEYS
            THEN #​​​​GP(0); FI;
    (* Check that only one encryption algorithm is requested for the KeyID and it is one of the activated algorithms *)
    IF TMP_KEY_PROGRAM_STRUCT.KEYID_CTRL.ENC_ALG does not set exactly one bit OR
        (TMP_KEY_PROGRAM_STRUCT.KEYID_CTRL.ENC_ALG & IA32_TME_ACTIVATE[63:48]) = 0
            THEN #​​​​GP(0); FI:
    Attempt to acquire lock to gain exclusive access to platform key table;
    IF attempt is unsuccessful
        THEN (* PCONFIG failure *)
            RFLAGS.ZF := 1;
            RAX := DEVICE_BUSY;
                    (* failure reason 5 *)
            GOTO EXIT;
    FI;
    CASE (TMP_KEY_PROGRAM_STRUCT.KEYID_CTRL.COMMAND) OF
        0 (KEYID_SET_KEY_DIRECT):
        Update TME-MK table for TMP_KEY_PROGRAM_STRUCT.KEYID as follows:
            Encrypt with the selected key
            Use the encryption algorithm selected by TMP_KEY_PROGRAM_STRUCT.KEYID_CTRL.ENC_ALG
            (* The number of bytes used by the next two lines depends on selected encryption algorithm *)
            DATA_KEY is TMP_KEY_PROGRAM_STRUCT.KEY_FIELD_1
            TWEAK_KEY is TMP_KEY_PROGRAM_STRUCT.KEY_FIELD_2
        BREAK;
        1 (KEYID_SET_KEY_RANDOM):
        Load TMP_RND_DATA_KEY with a random key using hardware RNG; (* key size depends on selected encryption algorithm *)
        IF there was insufficient entropy
            THEN (* PCONFIG failure *)
                RFLAGS.ZF := 1;
                RAX := ENTROPY_ERROR; (* failure reason 2 *)
                Release lock on platform key table;
                GOTO EXIT;
        FI;
        Load TMP_RND_TWEAK_KEY with a random key using hardware RNG; (* key size depends on selected encryption algorithm *)
        IF there was insufficient entropy
            THEN (* PCONFIG failure *)
                RFLAGS.ZF := 1;
                RAX := ENTROPY_ERROR; (* failure reason 2 *)
                Release lock on platform key table;
                GOTO EXIT;
        FI;
        (* Combine software-supplied entropy to the data key and tweak key *)
        (* The number of bytes used by the next two lines depends on selected encryption algorithm *)
        TMP_RND_DATA_KEY := TMP_RND_KEY XOR TMP_KEY_PROGRAM_STRUCT.KEY_FIELD_1;
        TMP_RND_TWEAK_KEY := TMP_RND_TWEAK_KEY XOR TMP_KEY_PROGRAM_STRUCT.KEY_FIELD_2;
        Update TME-MK table for TMP_KEY_PROGRAM_STRUCT.KEYID as follows:
            Encrypt with the selected key
            Use the encryption algorithm selected by TMP_KEY_PROGRAM_STRUCT.KEYID_CTRL.ENC_ALG
            (* The number of bytes used by the next two lines depends on selected encryption algorithm *)
            DATA_KEY is TMP_RND_DATA_KEY
            TWEAK_KEY is TMP_RND_TWEAK_KEY
        BREAK;
        2 (KEYID_CLEAR_KEY):
        Update TME-MK table for TMP_KEY_PROGRAM_STRUCT.KEYID as follows:
            Encrypt (or not) using the current configuration for TME
            The specified encryption algorithm and key values are not used.
        BREAK;
        3 (KEYID_NO_ENCRYPT):
        Update TME-MK table for TMP_KEY_PROGRAM_STRUCT.KEYID as follows:
            Do not encrypt
            The specified encryption algorithm and key values are not used.
        BREAK;
    ESAC;
    Release lock on platform key table;
ESAC;
RAX := 0;
RFLAGS.ZF := 0;
EXIT:
RFLAGS.CF := 0;
RFLAGS.PF := 0;
RFLAGS.AF := 0;
RFLAGS.OF := 0;
RFLAGS.SF := 0;

```

## Protected Mode Exceptions

| \#​​​​GP(0)                                                                                                                                                                                 | If input value in EAX encodes an unsupported leaf function. |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| If a memory operand effective address is outside the relevant segment limit.                                                                                                                |
| MKTME_KEY_PROGRAM leaf function:                                                                                                                                                            |
| If IA32_TME_ACTIVATE MSR is not locked.                                                                                                                                                     |
| If hardware encryption and TME-MK capability are not enabled in IA32_TME_ACTIVATE MSR.                                                                                                      |
| If the memory operand is not 256B aligned.                                                                                                                                                  |
| If any of the reserved bits in the KEYID_CTRL field of the MKTME_KEY_PROGRAM_STRUCT are set or that field indicates an unsupported KeyID, key-programming command, or encryption algorithm. |
| \#​PF(fault-code)                                                                                                                                                                           | If a page fault occurs in accessing memory operands.        |
| #​​​UD                                                                                                                                                                                      | If any of the LOCK/REP/Operand Size/VEX prefixes are used.  |
| If current privilege level is not 0.                                                                                                                                                        |
| If CPUID.7.0:EDX[bit 18] = 0                                                                                                                                                                |

## Real-Address Mode Exceptions

| \#​​​​GP                                                                                                                                                                                    | If input value in EAX encodes an unsupported leaf function. |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| MKTME_KEY_PROGRAM leaf function:                                                                                                                                                            |
| If IA32_TME_ACTIVATE MSR is not locked.                                                                                                                                                     |
| If hardware encryption and TME-MK capability are not enabled in IA32_TME_ACTIVATE MSR.                                                                                                      |
| If a memory operand is not 256B aligned.                                                                                                                                                    |
| If any of the reserved bits in the KEYID_CTRL field of the MKTME_KEY_PROGRAM_STRUCT are set or that field indicates an unsupported KeyID, key-programming command, or encryption algorithm. |
| #​​​UD                                                                                                                                                                                      | If any of the LOCK/REP/Operand Size/VEX prefixes are used.  |
| If current privilege level is not 0.                                                                                                                                                        |
| If CPUID.7.0:EDX.PCONFIG[bit 18] = 0                                                                                                                                                        |

## Virtual-8086 Mode Exceptions

| #​​​UD | PCONFIG instruction is not recognized in virtual-8086 mode. |
| ------ | ----------------------------------------------------------- |

## Compatibility Mode Exceptions

Same exceptions as in protected mode.

## 64-Bit Mode Exceptions

| \#​​​​GP(0)                                                                                                                                                                                 | If input value in EAX encodes an unsupported leaf function. |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| If a memory operand is non-canonical form.                                                                                                                                                  |
| MKTME_KEY_PROGRAM leaf function:                                                                                                                                                            |
| If IA32_TME_ACTIVATE MSR is not locked.                                                                                                                                                     |
| If hardware encryption and TME-MK capability are not enabled in IA32_TME_ACTIVATE MSR.                                                                                                      |
| If a memory operand is not 256B aligned.                                                                                                                                                    |
| If any of the reserved bits in the KEYID_CTRL field of the MKTME_KEY_PROGRAM_STRUCT are set or that field indicates an unsupported KeyID, key-programming command, or encryption algorithm. |
| \#​PF(fault-code)                                                                                                                                                                           | If a page fault occurs in accessing memory operands.        |
| #​​​UD                                                                                                                                                                                      | If any of the LOCK/REP/Operand Size/VEX prefixes are used.  |
| If the current privilege level is not 0.                                                                                                                                                    |
| If CPUID.7.0:EDX.PCONFIG[bit 18] = 0.                                                                                                                                                       |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
