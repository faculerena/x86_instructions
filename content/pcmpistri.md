# PCMPISTRI

**Packed Compare Implicit Length Strings**

| Opcode/Instruction                                            | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                                                           |
| ------------------------------------------------------------- | ----- | ---------------------- | ------------------ | --------------------------------------------------------------------------------------------------------------------- |
| 66 0F 3A 63 /r imm8 PCMPISTRI xmm1, xmm2/m128, imm8           | RM    | V/V                    | SSE4_2             | Perform a packed comparison of string data with implicit lengths, generating an index, and storing the result in ECX. |
| VEX.128.66.0F3A.WIG 63 /r ib VPCMPISTRI xmm1, xmm2/m128, imm8 | RM    | V/V                    | AVX                | Perform a packed comparison of string data with implicit lengths, generating an index, and storing the result in ECX. |

## Instruction Operand Encoding

| Op/En | Operand 1     | Operand 2     | Operand 3 | Operand 4 |
| ----- | ------------- | ------------- | --------- | --------- |
| RM    | ModRM:reg (r) | ModRM:r/m (r) | imm8      | N/A       |

## Description

The instruction compares data from two strings based on the encoded value in the imm8 control byte (see Section 4.1, “Imm8 Control Byte Operation for PCMPESTRI / PCMPESTRM / PCMPISTRI / PCMPISTRM”), and generates an index stored to ECX.

Each string is represented by a single value. The value is an xmm (or possibly m128 for the second operand) which contains the data elements of the string (byte or word data). Each input byte/word is augmented with a valid/invalid tag. A byte/word is considered valid only if it has a lower index than the least significant null byte/word. (The least significant null byte/word is also considered invalid.)

The comparison and aggregation operations are performed according to the encoded value of imm8 bit fields (see Section 4.1). The index of the first (or last, according to imm8[6]) set bit of IntRes2 is returned in ECX. If no bits are set in IntRes2, ECX is set to 16 (8).

Note that the Arithmetic Flags are written in a non-standard manner in order to supply the most relevant information:

CFlag – Reset if IntRes2 is equal to zero, set otherwise

ZFlag – Set if any byte/word of xmm2/mem128 is null, reset otherwise

SFlag – Set if any byte/word of xmm1 is null, reset otherwise

OFlag –IntRes2[0]

AFlag – Reset

PFlag – Reset

Note: In VEX.128 encoded version, VEX.vvvv is reserved and must be 1111b, VEX.L must be 0, otherwise the instruction will #​​​UD.

## Effective Operand Size

| Operating mode/size | Operand 1 | Operand 2 | Result |
| ------------------- | --------- | --------- | ------ |
| 16 bit              | xmm       | xmm/m128  | ECX    |
| 32 bit              | xmm       | xmm/m128  | ECX    |
| 64 bit              | xmm       | xmm/m128  | ECX    |

## Intel C/C++ Compiler Intrinsic Equivalent For Returning Index

int \_mm_cmpistri (\_\_m128i a, \_\_m128i b, const int mode);

## Intel C/C++ Compiler Intrinsics For Reading EFlag Results

int \_mm_cmpistra (\_\_m128i a, \_\_m128i b, const int mode);

int \_mm_cmpistrc (\_\_m128i a, \_\_m128i b, const int mode);

int \_mm_cmpistro (\_\_m128i a, \_\_m128i b, const int mode);

int \_mm_cmpistrs (\_\_m128i a, \_\_m128i b, const int mode);

int \_mm_cmpistrz (\_\_m128i a, \_\_m128i b, const int mode);

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
