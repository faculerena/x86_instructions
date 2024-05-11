# AESDECWIDE256KL

**Perform **

| Opcode/Instruction                                       | Op/En | 64/32-bit Mode | CPUID Feature Flag | Description                                                                                                                         |
| -------------------------------------------------------- | ----- | -------------- | ------------------ | ----------------------------------------------------------------------------------------------------------------------------------- |
| F3 0F 38 D8 !(11):011:bbb AESDECWIDE256KL m512, <XMM0-7> | A     | V/V            | AESKLEWIDE_KL      | Decrypt XMM0-7 using 256-bit AES key indicated by handle at m512 and store each resultant block back to its corresponding register. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1     | Operands 2—9           |
| ----- | ----- | ------------- | ---------------------- |
| A     | N/A   | ModRM:r/m (r) | Implicit XMM0-7 (r, w) |

## Description

The AESDECWIDE256KL1 instruction performs 14 rounds of AES to decrypt each of the eight blocks in XMM0-7 using the 256-bit key indicated by the handle from the second operand. It replaces each input block in XMM0-7 with its corresponding decrypted block if the operation succeeds (e.g., does not run into a handle violation failure).

## Operation

#### AESDECWIDE256KL

```
Handle := UnalignedLoad of 512 bit (SRC); // Load is not guaranteed to be atomic.
Illegal Handle = (HandleReservedBitSet (Handle) ||
                (Handle[0] AND (CPL > 0)) ||
                Handle [2] ||
                HandleKeyType (Handle) != HANDLE_KEY_TYPE_AES256);
IF (Illegal Handle) {
    THEN RFLAGS.ZF := 1;
    ELSE
        (UnwrappedKey, Authentic) := UnwrapKeyAndAuthenticate512 (Handle[511:0], IWKey);
        IF (Authentic == 0)
            THEN RFLAGS.ZF := 1;
            ELSE
                XMM0 := AES256Decrypt (XMM0, UnwrappedKey) ;
                XMM1 := AES256Decrypt (XMM1, UnwrappedKey) ;
                XMM2 := AES256Decrypt (XMM2, UnwrappedKey) ;
                XMM3 := AES256Decrypt (XMM3, UnwrappedKey) ;
                XMM4 := AES256Decrypt (XMM4, UnwrappedKey) ;
                XMM5 := AES256Decrypt (XMM5, UnwrappedKey) ;
                XMM6 := AES256Decrypt (XMM6, UnwrappedKey) ;
                XMM7 := AES256Decrypt (XMM7, UnwrappedKey) ;
                RFLAGS.ZF := 0;
        FI;
FI;
RFLAGS.OF, SF, AF, PF, CF := 0;

```

## Flags Affected

ZF is set to 0 if the operation succeeded and set to 1 if the operation failed due to a handle violation. The other arithmetic flags (OF, SF, AF, PF, CF) are cleared to 0.

1. Further details on Key Locker and usage of this instruction can be found here:

### https://software.intel.com/content/www/us/en/develop/download/intel-key-locker-specification.html.

## Intel C/C++ Compiler Intrinsic Equivalent

```
AESDECWIDE256KLunsigned char _mm_aesdecwide256kl_u8(__m128i odata[8], const __m128i idata[8], const void* h);

```

## Exceptions (All Operating Modes)

#​​​UD If the LOCK prefix is used.

If CPUID.07H:ECX.KL[bit 23] = 0.

If CR4.KL = 0.

If CPUID.19H:EBX.AESKLE[bit 0] = 0.

If CR0.EM = 1.

If CR4.OSFXSR = 0.

If CPUID.19H:EBX.WIDE_KL[bit 2] = 0.

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
