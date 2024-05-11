# VCOMISH

**Compare Scalar Ordered FP**

| **Description**EVEX.LLIG.NP.MAP5.W0 2F /r A V/V AVX512-FP16 VCOMISH xmm1, xmm2/m16 {sae} **Description**EVEX.LLIG.NP.MAP5.W0 2F /r A V/V AVX512-FP16 **p/ 64/32 CPUID Feature Instruction En Bit Mode Flag Support Description**EVEX.LLIG.NP.MAP5.W0 2F /r A V/V AVX512-FP16 **Description**EVEX.LLIG.NP.MAP5.W0 2F /r A V/V AVX512-FP16 **Instruction En Bit Mode Flag Support Instruction En Bit Mode Flag Support 64/32 CPUID Feature Instruction En Bit Mode Flag CPUID Feature Instruction En Bit Mode Flag Op/ 64/32 CPUID Feature Instruction En Bit Mode Flag 64/32 CPUID Feature Instruction En Bit Mode Flag CPUID Feature Instruction En Bit Mode Flag Op/ 64/32 CPUID Feature** |     | **Support** |     | **Description**                                                                     |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- | ----------- | --- | ----------------------------------------------------------------------------------- |
| VCOMISH xmm1, xmm2/m16 {sae}                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |     |             |     | Compare low FP16 values in xmm1 and xmm2/m16, and set the EFLAGS flags accordingly. |

## Instruction Operand Encoding

| Op/En | Tuple  | Operand 1     | Operand 2     | Operand 3 | Operand 4 |
| ----- | ------ | ------------- | ------------- | --------- | --------- |
| A     | Scalar | ModRM:reg (r) | ModRM:r/m (r) | N/A       | N/A       |

### Description

This instruction compares the FP16 values in the low word of operand 1 (first operand) and operand 2 (second operand), and sets the ZF, PF, and CF flags in the EFLAGS register according to the result (unordered, greater than, less than, or equal). The OF, SF and AF flags in the EFLAGS register are set to 0. The unordered result is returned if either source operand is a NaN (QNaN or SNaN).

Operand 1 is an XMM register; operand 2 can be an XMM register or a 16-bit memory location.

The VCOMISH instruction differs from the VUCOMISH instruction in that it signals a SIMD floating-point invalid operation exception (#​I) when a source operand is either a QNaN or SNaN. The VUCOMISH instruction signals an invalid numeric exception only if a source operand is an SNaN.

The EFLAGS register is not updated if an unmasked SIMD floating-point exception is generated. EVEX.vvvv is reserved and must be 1111b, otherwise instructions will #​​​UD.

### Operation

#### VCOMISH SRC1, SRC2

```
RESULT := OrderedCompare(SRC1.fp16[0],SRC2.fp16[0])
IF RESULT is UNORDERED:
    ZF, PF, CF := 1, 1, 1
ELSE IF RESULT is GREATER_THAN:
    ZF, PF, CF := 0, 0, 0
ELSE IF RESULT is LESS_THAN:
    ZF, PF, CF := 0, 0, 1
ELSE: // RESULT is EQUALS
    ZF, PF, CF := 1, 0, 0
OF, AF, SF := 0, 0, 0

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
VCOMISH int _mm_comi_round_sh (__m128h a, __m128h b, const int imm8, const int sae);

```

```
VCOMISH int _mm_comi_sh (__m128h a, __m128h b, const int imm8);

```

```
VCOMISH int _mm_comieq_sh (__m128h a, __m128h b);

```

```
VCOMISH int _mm_comige_sh (__m128h a, __m128h b);

```

```
VCOMISH int _mm_comigt_sh (__m128h a, __m128h b);

```

```
VCOMISH int _mm_comile_sh (__m128h a, __m128h b);

```

```
VCOMISH int _mm_comilt_sh (__m128h a, __m128h b);

```

```
VCOMISH int _mm_comineq_sh (__m128h a, __m128h b);

```

### SIMD Floating-Point Exceptions

Invalid, Denormal.

### Other Exceptions

EVEX-encoded instructions, see Table 2-48, “Type E3NF Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
