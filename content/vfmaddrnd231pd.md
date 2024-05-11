# VFMADDRND231PD

**Fused Multiply**

| Opcode/ Mode CPUID Description Instruction Support Feature Flag VEX.DDS.128.66.0F3A.W1 B8 /r /ib V/V FMA Multiply packed double-precision floating-point values from xmm1 VFMADDRND231PD xmm0, and xmm2/mem, add to xmm0 and xmm1, xmm2/m128, imm8 put result in xmm0. VEX.DDS.256.66.0F3A.W1 B8 /r /ib V/V FMA Multiply packed double-precision floating-point values from ymm1 VFMADDRND231PD ymm0, and ymm2/mem, add to ymm0 and ymm1, ymm2/m256, imm8 put result in ymm0. |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

## Description

Multiplies the two or four packed double-precision floating-point values from the second source operand to the two or four packed double-precision floating-point values in the third source operand, adds the infinite precision intermediate result to the two or four packed double-precision floating-point values in the first source operand, performs rounding and stores the resulting two or four packed double-precision floating-point values to the destination operand (first source operand).

The immediate byte defines several bit fields that control rounding, DAZ, FTZ, and exception suppression (See[Table 5-3](/x86/vfmaddrnd231pd#tbl-5-3)).The rounding mode specified in MXCSR.RC may be bypassed if the immediate bit called MS1 (MXCSR.RC Override) is set. Likewise, the MXCSR.FTZ and MXCSR.DAZ may also be bypassed if the immediate bit called MS2 (MXCSR.FTZ/DAZ Override) is set. In case SAE (Suppress All Exceptions) bit is set (i.e. imm8[3] = 1), the status flags in MXCSR are not updated and no SIMD floating-point exceptions are raised. When SAE bit is not set (i.e. imm8[3] = 0) then SIMD floating-point exceptions are signaled according to the MXCSR. If any result operand is an SNaN then it will be converted to a QNaN.

VEX.256 encoded version: The destination operand (also first source operand) is a YMM register and encoded in reg_field. The second source operand is a YMM register and encoded in VEX.vvvv. The third source operand is a YMM register or a 256-bit memory location and encoded in rm_field.

VEX.128 encoded version: The destination operand (also first source operand) is a XMM register and encoded in reg_field. The second source operand is a XMM register and encoded in VEX.vvvv. The third source operand is a XMM register or a 128-bit memory location and encoded in rm_field. The upper 128 bits of the YMM destination register are zeroed.

| Bits       | Field Name/value                           | Description                                                  | Comment        |
| ---------- | ------------------------------------------ | ------------------------------------------------------------ | -------------- |
| Imm8[1:0 ] | RC=0                                       | Round to nearest even                                        | If Imm8[2] = 1 |
| RC=1       | Round down                                 |
| RC=2       | Round up                                   |
| RC=3       | Truncate                                   |
| Imm8[2]    | MS1=0                                      | Use MXCSR.RC for rounding                                    |                |
| MS1=1      | Use Imm8[1:0] for rounding                 | Ignore MXCSR.RC                                              |
| Imm8[3]    | SAE=0                                      | Use MXCSR Exception Mask settings                            |                |
| SAE=1      | Suppress all Exception signaling           | Numerical result is computed as if FP exceptions are masked. |
| Imm8[4]    | MS2=0                                      | Use MXCSR.DAZ and MXCSR.FTZ                                  |                |
| MS2=1      | Use Imm8[6:5] to control DAZ/FTZ operation | Ignore MXCSR.DAZ and MXCSR.FTZ                               |
| Imm8[5]    | DAZ                                        | Control DAZ                                                  | IF MS2 = 1     |
| Imm8[6]    | FTZ                                        | Control FTZ                                                  | IF MS2 = 1     |
| Imm8[7]    | MBZ                                        | Must be zero                                                 |                |

[Table 5-3](/x86/vfmaddrnd231pd#tbl-5-3). Immediate Byte Encoding

Compiler tools may optionally support the complementary mnemonic VMADDRND321PD. The behavior of the complementary mnemonic in situations involving NANs are governed by the definition of the instruction mnemonic defined in the opcode/instruction column. See also Section 2.3.1, “FMA Instruction Operand Order and Arithmetic Behavior”

## Operation

```
In the operations below, “+” and “*” symbols represent multiplication and addition with infinite precision inputs and outputs (no rounding)

```

### VFMADDRND231PD DEST, SRC2, SRC3, imm8

```
IF (VEX.128) THEN
    MAXVL =2
ELSEIF (VEX.256)
    MAXVL = 4
FI
IF (imm8[3] = 1) THEN
    Suppress_SIMD_Exception_Signaling_Reporting();
FI
For i = 0 to MAXVL-1 {
    n = 64*i;
    DEST[n+63:n]←RoundFPControl_Imm((SRC2[n+63:n]*SRC3[n+63:n] + DEST[n+63:n]), imm8)
}
IF (VEX.128) THEN
DEST[255:128] ← 0
FI
IF (imm8[3] = 1) THEN
    Resume_SIMD_Exception_Signaling_Reporting();
FI

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
VFMADDRND231PD __m128d _mm_fmaddround_pd (__m128d a, __m128d b, __m128d c, const int ctrl);

```

```
VFMADDRND231PD __m256d _mm256_fmaddround_pd (__m256d a, __m256d b, __m256d c, const int ctrl);

```

## SIMD Floating-Point Exceptions

IF imm[3] = 1 Then

None

Else

Overflow, Underflow, Invalid, Precision, Denormal

FI

## Other Exceptions

See Exceptions Type 2

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
