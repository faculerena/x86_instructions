# ENCODEKEY128

**Encode **

| Opcode/Instruction                                               | Op/En | 64/32-bit Mode | CPUID Feature Flag | Description                                                                     |
| ---------------------------------------------------------------- | ----- | -------------- | ------------------ | ------------------------------------------------------------------------------- |
| F3 0F 38 FA 11:rrr:bbb ENCODEKEY128 r32, r32, <XMM0-2>, <XMM4-6> | A     | V/V            | AESKLE             | Wrap a 128-bit AES key from XMM0 into a key handle and output handle in XMM0—2. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1     | Operand 2     | Operand 3            | Operands 4—5        | Operands 6—7        |
| ----- | ----- | ------------- | ------------- | -------------------- | ------------------- | ------------------- |
| A     | N/A   | ModRM:reg (w) | ModRM:r/m (r) | Implicit XMM0 (r, w) | Implicit XMM1—2 (w) | Implicit XMM4—6 (w) |

## Description

The ENCODEKEY1281 instruction wraps a 128-bit AES key from the implicit operand XMM0 into a key handle that is then stored in the implicit destination operands XMM0-2.

The explicit source operand specifies handle restrictions, if any.

The explicit destination operand is populated with information on the source of the key and its attributes. XMM4 through XMM6 are reserved for future usages and software should not rely upon them being zeroed.

## Operation

#### ENCODEKEY128

```
#​​​​GP (0) if a reserved bit2 in SRC[31:0] is set
InputKey[127:0] := XMM0;
KeyMetadata[2:0] = SRC[2:0];
KeyMetadata[23:3] = 0;
    // Reserved for future usage
KeyMetadata[27:24] = 0;
    // KeyType is AES-128 (value of 0)
KeyMetadata[127:28] = 0;
    // Reserved for future usage
// KeyMetadata is the AAD input and InputKey is the Plaintext input for WrapKey128
Handle[383:0] := WrapKey128(InputKey[127:0], KeyMetadata[127:0], IWKey.Integrity Key[127:0], IWKey.Encryption Key[255:0]);
DEST[0] := IWKey.NoBackup;
DEST[4:1] := IWKey.KeySource[3:0];
DEST[31:5] = 0;
XMM0 := Handle[127:0]; // AAD
XMM1 := Handle[255:128]; // Integrity Tag
XMM2 := Handle[383:256]; // CipherText
/
XMM4 := 0;
/
XMM4 := 0;
R
XMM4 := 0;
e
XMM4 := 0;
s
XMM4 := 0;
e
XMM4 := 0;
r
XMM4 := 0;
v
XMM4 := 0;
e
XMM4 := 0;
d
XMM4 := 0;
f
XMM4 := 0;
o
XMM4 := 0;
r
XMM4 := 0;
f
XMM4 := 0;
u
XMM4 := 0;
t
XMM4 := 0;
u
XMM4 := 0;
r
XMM4 := 0;
e
XMM4 := 0;
XMM4 := 0;
/
XMM5 := 0;
/
XMM5 := 0;
R
XMM5 := 0;
e
XMM5 := 0;
s
XMM5 := 0;
e
XMM5 := 0;
r
XMM5 := 0;
v
XMM5 := 0;
e
XMM5 := 0;
d
XMM5 := 0;
f
XMM5 := 0;
o
XMM5 := 0;
r
XMM5 := 0;
f
XMM5 := 0;
u
XMM5 := 0;
t
XMM5 := 0;
u
XMM5 := 0;
r
XMM5 := 0;
e
XMM5 := 0;
XMM5 := 0;
/
XMM6 := 0;
/
XMM6 := 0;
R
XMM6 := 0;
e
XMM6 := 0;
s
XMM6 := 0;
e
XMM6 := 0;
r
XMM6 := 0;
v
XMM6 := 0;
e
XMM6 := 0;
d
XMM6 := 0;
f
XMM6 := 0;
o
XMM6 := 0;
r
XMM6 := 0;
f
XMM6 := 0;
u
XMM6 := 0;
t
XMM6 := 0;
u
XMM6 := 0;
r
XMM6 := 0;
e
XMM6 := 0;
XMM6 := 0;
RFLAGS.OF, SF, ZF, AF, PF, CF := 0;

```

> 2. SRC[31:3] are currently reserved for future usages. SRC[2], which indicates a no-decrypt restriction, is reserved if CPUID.19H:EAX[2] is 0. SRC[1], which indicates a no-encrypt restriction, is reserved if CPUID.19H:EAX[1] is 0. SRC[0], which indicates a CPL0-only restriction, is reserved if CPUID.19H:EAX[0] is 0.

## Flags Affected

All arithmetic flags (OF, SF, ZF, AF, PF, CF) are cleared to 0. Although they are cleared for the currently defined operations, future extensions may report information in the flags.

1. Further details on Key Locker and usage of this instruction can be found here:

### https://software.intel.com/content/www/us/en/develop/download/intel-key-locker-specification.html.

## Intel C/C++ Compiler Intrinsic Equivalent

```
ENCODEKEY128 unsigned int _mm_encodekey128_u32(unsigned int htype, __m128i key, void* h);

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
