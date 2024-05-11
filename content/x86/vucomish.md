# VUCOMISH**Unordered Compare Scalar FP**

| **Instruction En bit Mode Flag Support Instruction En bit Mode Flag Support 64/32 CPUID Feature Instruction En bit Mode Flag CPUID Feature Instruction En bit Mode Flag Op/ 64/32 CPUID Feature Instruction En bit Mode Flag 64/32 CPUID Feature Instruction En bit Mode Flag CPUID Feature Instruction En bit Mode Flag p/ 64/32 CPUID Feature Instruction En bit Mode Flag Support Description**EVEX.LLIG.NP.MAP5.W0 2E /r A V/V AVX512-FP16 **Description**EVEX.LLIG.NP.MAP5.W0 2E /r A V/V AVX512-FP16 VUCOMISH xmm1, xmm2/m16 {sae} **Description**EVEX.LLIG.NP.MAP5.W0 2E /r A V/V AVX512-FP16 **Description**EVEX.LLIG.NP.MAP5.W0 2E /r A V/V AVX512-FP16 **Op/ 64/32 CPUID Feature** |     | **Support** |     | **Description**                                                                    |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- | ----------- | --- | ---------------------------------------------------------------------------------- |
| VUCOMISH xmm1, xmm2/m16 {sae}                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |     |             |     | Compare low FP16 values in xmm1 and xmm2/m16 and set the EFLAGS flags accordingly. |

## Instruction Operand Encoding

| Op/En | Tuple  | Operand 1     | Operand 2     | Operand 3 | Operand 4 |
| ----- | ------ | ------------- | ------------- | --------- | --------- |
| A     | Scalar | ModRM:reg (w) | ModRM:r/m (r) | N/A       | N/A       |

### Description

This instruction compares the FP16 values in the low word of operand 1 (first operand) and operand 2 (second operand), and sets the ZF, PF, and CF flags in the EFLAGS register according to the result (unordered, greater than, less than, or equal). The OF, SF and AF flags in the EFLAGS register are set to 0. The unordered result is returned if either source operand is a NaN (QNaN or SNaN).

Operand 1 is an XMM register; operand 2 can be an XMM register or a 16-bit memory location.

The VUCOMISH instruction differs from the VCOMISH instruction in that it signals a SIMD floating-point invalid operation exception (#​I) only if a source operand is an SNaN. The COMISS instruction signals an invalid numeric exception when a source operand is either a QNaN or SNaN.

The EFLAGS register is not updated if an unmasked SIMD floating-point exception is generated. EVEX.vvvv are reserved and must be 1111b, otherwise instructions will #​​​UD.

### Operation

#### VUCOMISH

```
RESULT := UnorderedCompare(SRC1.fp16[0],SRC2.fp16[0])
if RESULT is UNORDERED:
    ZF, PF, CF := 1, 1, 1
else if RESULT is GREATER_THAN:
    ZF, PF, CF := 0, 0, 0
else if RESULT is LESS_THAN:
    ZF, PF, CF := 0, 0, 1
else: // RESULT is EQUALS
    ZF, PF, CF := 1, 0, 0
OF, AF, SF := 0, 0, 0

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
VUCOMISH int _mm_ucomieq_sh (__m128h a, __m128h b);

```

```
VUCOMISH int _mm_ucomige_sh (__m128h a, __m128h b);

```

```
VUCOMISH int _mm_ucomigt_sh (__m128h a, __m128h b);

```

```
VUCOMISH int _mm_ucomile_sh (__m128h a, __m128h b);

```

```
VUCOMISH int _mm_ucomilt_sh (__m128h a, __m128h b);

```

```
VUCOMISH int _mm_ucomineq_sh (__m128h a, __m128h b);

```

### SIMD Floating-Point Exceptions

Invalid, Denormal.

### Other Exceptions

EVEX-encoded instructions, see Table 2-48, “Type E3NF Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
