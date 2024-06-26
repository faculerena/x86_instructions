# SHA1RNDS4

**Perform Four Rounds of SHA**

| Opcode/Instruction                                | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                                                                                                                                                                                          |
| ------------------------------------------------- | ----- | ---------------------- | ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| NP 0F 3A CC /r ib SHA1RNDS4 xmm1, xmm2/m128, imm8 | RMI   | V/V                    | SHA                | Performs four rounds of SHA1 operation operating on SHA1 state (A,B,C,D) from xmm1, with a pre-computed sum of the next 4 round message dwords and state variable E from xmm2/m128. The immediate byte controls logic functions and round constants. |

## Instruction Operand Encoding

| Op/En | Operand 1        | Operand 2     | Operand 3 |
| ----- | ---------------- | ------------- | --------- |
| RMI   | ModRM:reg (r, w) | ModRM:r/m (r) | imm8      |

## Description

The SHA1RNDS4 instruction performs four rounds of SHA1 operation using an initial SHA1 state (A,B,C,D) from the first operand (which is a source operand and the destination operand) and some pre-computed sum of the next 4 round message dwords, and state variable E from the second operand (a source operand). The updated SHA1 state (A,B,C,D) after four rounds of processing is stored in the destination operand.

## Operation

### SHA1RNDS4

```
The function f() and Constant K are dependent on the value of the immediate.
IF ( imm8[1:0] = 0 )
    THEN f() := f0(),
        K := K0;
ELSE IF ( imm8[1:0]
        = 1 )
    THEN f() := f1(),
        K := K1;
ELSE IF ( imm8[1:0]
        = 2 )
    THEN f() := f2(),
        K := K2;
ELSE IF ( imm8[1:0]
        = 3 )
    THEN f() := f3(),
        K := K3;
FI;
A := SRC1[127:96];
B := SRC1[95:64];
C := SRC1[63:32];
D := SRC1[31:0];
W0E := SRC2[127:96];
W1 := SRC2[95:64];
W2 := SRC2[63:32];
W3 := SRC2[31:0];
Round i = 0 operation:
A_1 := f (B, C, D) + (A ROL 5) +W0E +K;
B_1 := A;
C_1 := B ROL 30;
D_1 := C;
E_1 := D;
FOR i = 1 to 3
    A_(i +1) := f (B_i, C_i, D_i) + (A_i ROL 5) +Wi+ E_i +K;
    B_(i +1) := A_i;
    C_(i +1) := B_i ROL 30;
    D_(i +1) := C_i;
    E_(i +1) := D_i;
ENDFOR
DEST[127:96] := A_4;
DEST[95:64] := B_4;
DEST[63:32] := C_4;
DEST[31:0] := D_4;

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
SHA1RNDS4 __m128i _mm_sha1rnds4_epu32(__m128i, __m128i, const int);

```

## Flags Affected

None.

## SIMD Floating-Point Exceptions

None.

## Other Exceptions

See Table 2-21, “Type 4 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
