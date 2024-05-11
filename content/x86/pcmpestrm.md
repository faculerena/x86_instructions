# PCMPESTRM

**Packed Compare Explicit Length Strings**

| Opcode/Instruction                                        | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                                                          |
| --------------------------------------------------------- | ----- | ---------------------- | ------------------ | -------------------------------------------------------------------------------------------------------------------- |
| 66 0F 3A 60 /r imm8 PCMPESTRM xmm1, xmm2/m128, imm8       | RMI   | V/V                    | SSE4_2             | Perform a packed comparison of string data with explicit lengths, generating a mask, and storing the result in XMM0. |
| VEX.128.66.0F3A 60 /r ib VPCMPESTRM xmm1, xmm2/m128, imm8 | RMI   | V/V                    | AVX                | Perform a packed comparison of string data with explicit lengths, generating a mask, and storing the result in XMM0. |

## Instruction Operand Encoding

| Op/En | Operand 1     | Operand 2     | Operand 3 | Operand 4 |
| ----- | ------------- | ------------- | --------- | --------- |
| RMI   | ModRM:reg (r) | ModRM:r/m (r) | imm8      | N/A       |

## Description

The instruction compares data from two string fragments based on the encoded value in the imm8 contol byte (see Section 4.1, “Imm8 Control Byte Operation for PCMPESTRI / PCMPESTRM / PCMPISTRI / PCMPISTRM”), and generates a mask stored to XMM0.

Each string fragment is represented by two values. The first value is an xmm (or possibly m128 for the second operand) which contains the data elements of the string (byte or word data). The second value is stored in an input length register. The input length register is EAX/RAX (for xmm1) or EDX/RDX (for xmm2/m128). The length represents the number of bytes/words which are valid for the respective xmm/m128 data.

The length of each input is interpreted as being the absolute-value of the value in the length register. The absolute-value computation saturates to 16 (for bytes) and 8 (for words), based on the value of imm8[bit3] when the value in the length register is greater than 16 (8) or less than -16 (-8).

The comparison and aggregation operations are performed according to the encoded value of imm8 bit fields (see Section 4.1). As defined by imm8[6], IntRes2 is then either stored to the least significant bits of XMM0 (zero extended to 128 bits) or expanded into a byte/word-mask and then stored to XMM0.

Note that the Arithmetic Flags are written in a non-standard manner in order to supply the most relevant information:

CFlag – Reset if IntRes2 is equal to zero, set otherwise

ZFlag – Set if absolute-value of EDX is < 16 (8), reset otherwise

SFlag – Set if absolute-value of EAX is < 16 (8), reset otherwise

OFlag –IntRes2[0]

AFlag – Reset

PFlag – Reset

Note: In VEX.128 encoded versions, bits (MAXVL-1:128) of XMM0 are zeroed. VEX.vvvv is reserved and must be 1111b, VEX.L must be 0, otherwise the instruction will #​​​UD.

## Effective Operand Size

| Operating mode/size | Operand 1 | Operand 2 | Length 1 | Length 2 | Result |
| ------------------- | --------- | --------- | -------- | -------- | ------ |
| 16 bit              | xmm       | xmm/m128  | EAX      | EDX      | XMM0   |
| 32 bit              | xmm       | xmm/m128  | EAX      | EDX      | XMM0   |
| 64 bit              | xmm       | xmm/m128  | EAX      | EDX      | XMM0   |
| 64 bit + REX.W      | xmm       | xmm/m128  | RAX      | RDX      | XMM0   |

## Intel C/C++ Compiler Intrinsic Equivalent For Returning Mask

\_\_m128i \_mm_cmpestrm (\_\_m128i a, int la, \_\_m128i b, int lb, const int mode);

## Intel C/C++ Compiler Intrinsics For Reading EFlag Results

int \_mm_cmpestra (\_\_m128i a, int la, \_\_m128i b, int lb, const int mode);

int \_mm_cmpestrc (\_\_m128i a, int la, \_\_m128i b, int lb, const int mode);

int \_mm_cmpestro (\_\_m128i a, int la, \_\_m128i b, int lb, const int mode);

int \_mm_cmpestrs (\_\_m128i a, int la, \_\_m128i b, int lb, const int mode);

int \_mm_cmpestrz (\_\_m128i a, int la, \_\_m128i b, int lb, const int mode);

## SIMD Floating-Point Exceptions

None.

## Other Exceptions

See Table 2-21, “Type 4 Class Exception Conditions,” additionally, this instruction does not cause #​​​​GP if the memory operand is not aligned to 16 Byte boundary, and:

| #​​​UD               | If VEX.L = 1. |
| -------------------- | ------------- |
| If VEX.vvvv ≠ 1111B. |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
