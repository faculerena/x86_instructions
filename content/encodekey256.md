#ENCODEKEY256
**Encode **

| Opcode/Instruction                                    | Op/En | 64/32-bit Mode | CPUID Feature Flag | Description                                                                     |
| ----------------------------------------------------- | ----- | -------------- | ------------------ | ------------------------------------------------------------------------------- |
| F3 0F 38 FB 11:rrr:bbb ENCODEKEY256 r32, r32 <XMM0-6> | A     | V/V            | AESKLE             | Wrap a 256-bit AES key from XMM1:XMM0 into a key handle and store it in XMM0—3. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1     | Operand 2     | Operands 3—4           | Operands 5—9        |
| ----- | ----- | ------------- | ------------- | ---------------------- | ------------------- |
| A     | N/A   | ModRM:reg (w) | ModRM:r/m (r) | Implicit XMM0—1 (r, w) | Implicit XMM2—6 (w) |

## Description

The ENCODEKEY2561 instruction wraps a 256-bit AES key from the implicit operand XMM1:XMM0 into a key handle that is then stored in the implicit destination operands XMM0-3.

The explicit source operand is a general-purpose register and specifies what handle restrictions should be built into the handle.

The explicit destination operand is populated with information on the source of the key and its attributes. XMM4 through XMM6 are reserved for future usages and software should not rely upon them being zeroed.

## Operation

#### ENCODEKEY256

```
#​​​​GP (0) if a reserved bit2 in SRC[31:0] is set
InputKey[255:0] := XMM1:XMM0;
KeyMetadata[2:0] = SRC[2:0];
KeyMetadata[23:3] = 0; // Reserved for future usage
KeyMetadata[27:24] = 1; // KeyType is AES-256 (value of 1)
KeyMetadata[127:28] = 0; // Reserved for future usage
// KeyMetadata is the AAD input and InputKey is the Plaintext input for WrapKey256
Handle[511:0] := WrapKey256(InputKey[255:0], KeyMetadata[127:0], IWKey.Integrity Key[127:0], IWKey.Encryption Key[255:0]);
DEST[0] := IWKey.NoBackup;
DEST[4:1] := IWKey.KeySource[3:0];
DEST[31:5] = 0;
XMM0 := Handle[127:0]; // AAD
XMM1 := Handle[255:128]; // Integrity Tag
XMM2 := Handle[383:256]; // CipherText[127:0]
XMM3 := Handle[511:384]; // CipherText[255:128]
XMM4 := 0;
    // Reserved for future usage
XMM5 := 0;
    // Reserved for future usage
XMM6 := 0;
    // Reserved for future usage
RFLAGS.OF, SF, ZF, AF, PF, CF := 0;
1. Further details on Key Locker and usage of this instruction can be found here:

```

### https://software.intel.com/content/www/us/en/develop/download/intel-key-locker-specification.html.

2. SRC[31:3] are currently reserved for future usages. SRC[2], which indicates a no-decrypt restriction, is reserved if CPUID.19H:EAX[2] is 0. SRC[1], which indicates a no-encrypt restriction, is reserved if CPUID.19H:EAX[1] is 0. SRC[0], which indicates a CPL0-only restriction, is reserved if CPUID.19H:EAX[0] is 0.

## Flags Affected

All arithmetic flags (OF, SF, ZF, AF, PF, CF) are cleared to 0. Although they are cleared for the currently defined operations, future extensions may report information in the flags.

## Intel C/C++ Compiler Intrinsic Equivalent

```
ENCODEKEY256 unsigned int _mm_encodekey256_u32(unsigned int htype, __m128i key_lo, __m128i key_hi, void* h);

```

## Exceptions (All Operating Modes)

#​​​​GP If reserved bit is set in source register value.

#​​​UD If the LOCK prefix is used.

If CPUID.07H:ECX.KL[bit 23] = 0.

If CR4.KL = 0.

If CPUID.19H:EBX.AESKLE[bit 0] = 0.

If CR0.EM = 1.

If CR4.OSFXSR = 0.

#​NM If CR0.TS = 1.

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
