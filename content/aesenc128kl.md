# AESENC128KL

**Perform Ten Rounds of AES Encryption Flow With Key Locker Using **

| Opcode/Instruction                              | Op/En | 64/32-bit Mode | CPUID Feature Flag | Description                                                                            |
| ----------------------------------------------- | ----- | -------------- | ------------------ | -------------------------------------------------------------------------------------- |
| F3 0F 38 DC !(11):rrr:bbb AESENC128KL xmm, m384 | A     | V/V            | AESKLE             | Encrypt xmm using 128-bit AES key indicated by handle at m384 and store result in xmm. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1        | Operand 2     | Operand 3 | Operand 4 |
| ----- | ----- | ---------------- | ------------- | --------- | --------- |
| A     | N/A   | ModRM:reg (r, w) | ModRM:r/m (r) | N/A       | N/A       |

## Description

The AESENC128KL1 instruction performs ten rounds of AES to encrypt the first operand using the 128-bit key indicated by the handle from the second operand. It stores the result in the first operand if the operation succeeds (e.g., does not run into a handle violation failure).

## Operation

#### AESENC128KL

```
Handle := UnalignedLoad of 384 bit (SRC); // Load is not guaranteed to be atomic.
Illegal Handle = (
                HandleReservedBitSet (Handle) ||
                (Handle[0] AND (CPL > 0)) ||
                Handle [1] ||
                HandleKeyType (Handle) != HANDLE_KEY_TYPE_AES128
                );
IF (Illegal Handle) {
    THEN RFLAGS.ZF := 1;
    ELSE
        (UnwrappedKey, Authentic) := UnwrapKeyAndAuthenticate384 (Handle[383:0], IWKey);
        IF (Authentic == 0)
        THEN RFLAGS.ZF := 1;
        ELSE
            DEST := AES128Encrypt (DEST, UnwrappedKey) ;
            RFLAGS.ZF := 0;
        FI;
FI;
RFLAGS.OF, SF, AF, PF, CF := 0;

```

## Flags Affected

ZF is set to 0 if the operation succeeded and set to 1 if the operation failed due to a handle violation. The other arithmetic flags (OF, SF, AF, PF, CF) are cleared to 0.

## Intel C/C++ Compiler Intrinsic Equivalent

```
AESENC128KL unsigned char _mm_aesenc128kl_u8(__m128i* odata, __m128i idata, const void* h);

```

```
1. Further details on Key Locker and usage of this instruction can be found here:

```

### https://software.intel.com/content/www/us/en/develop/download/intel-key-locker-specification.html.

## Exceptions (All Operating Modes)

#​​​UD If the LOCK prefix is used.

If CPUID.07H:ECX.KL[bit 23] = 0.

If CR4.KL = 0.

If CPUID.19H:EBX.AESKLE[bit 0] = 0.

If CR0.EM = 1.

If CR4.OSFXSR = 0.

#​NM If CR0.TS = 1.

#​PF If a page fault occurs.

#​​​​GP(0) If a memory operand effective address is outside the CS, DS, ES, FS, or GS segment limit.

If the DS, ES, FS, or GS register is used to access memory and it contains a NULL segment selector.

If the memory address is in a non-canonical form.

#​​​​​SS(0) If a memory operand effective address is outside the SS segment limit.

If a memory address referencing the SS segment is in a non-canonical form.

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
