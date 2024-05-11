# MOVNTPS

**Store Packed Single Precision Floating**

| Opcode/Instruction                       | Op / En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                  |
| ---------------------------------------- | ------- | ---------------------- | ------------------ | ---------------------------------------------------------------------------- |
| NP 0F 2B /r MOVNTPS m128, xmm1           | A       | V/V                    | SSE                | Move packed single precision values xmm1 to mem using non-temporal hint.     |
| VEX.128.0F.WIG 2B /r VMOVNTPS m128, xmm1 | A       | V/V                    | AVX                | Move packed single precision values xmm1 to mem using non-temporal hint.     |
| VEX.256.0F.WIG 2B /r VMOVNTPS m256, ymm1 | A       | V/V                    | AVX                | Move packed single precision values ymm1 to mem using non-temporal hint.     |
| EVEX.128.0F.W0 2B /r VMOVNTPS m128, xmm1 | B       | V/V                    | AVX512VL AVX512F   | Move packed single precision values in xmm1 to m128 using non-temporal hint. |
| EVEX.256.0F.W0 2B /r VMOVNTPS m256, ymm1 | B       | V/V                    | AVX512VL AVX512F   | Move packed single precision values in ymm1 to m256 using non-temporal hint. |
| EVEX.512.0F.W0 2B /r VMOVNTPS m512, zmm1 | B       | V/V                    | AVX512F            | Move packed single precision values in zmm1 to m512 using non-temporal hint. |

## Instruction Operand Encoding1

> 1. ModRM.MOD != 011B

| Op/En | Tuple Type | Operand 1     | Operand 2     | Operand 3 | Operand 4 |
| ----- | ---------- | ------------- | ------------- | --------- | --------- |
| A     | N/A        | ModRM:r/m (w) | ModRM:reg (r) | N/A       | N/A       |
| B     | Full Mem   | ModRM:r/m (w) | ModRM:reg (r) | N/A       | N/A       |

## Description

Moves the packed single precision floating-point values in the source operand (second operand) to the destination operand (first operand) using a non-temporal hint to prevent caching of the data during the write to memory. The source operand is an XMM register, YMM register or ZMM register, which is assumed to contain packed single precision, floating-pointing. The destination operand is a 128-bit, 256-bit or 512-bit memory location. The memory operand must be aligned on a 16-byte (128-bit version), 32-byte (VEX.256 encoded version) or 64-byte (EVEX.512 encoded version) boundary otherwise a general-protection exception (#​​​​GP) will be generated.

The non-temporal hint is implemented by using a write combining (WC) memory type protocol when writing the data to memory. Using this protocol, the processor does not write the data into the cache hierarchy, nor does it fetch the corresponding cache line from memory into the cache hierarchy. The memory type of the region being written to can override the non-temporal hint, if the memory address specified for the non-temporal store is in an uncacheable (UC) or write protected (WP) memory region. For more information on non-temporal stores, see “Caching of Temporal vs. Non-Temporal Data” in Chapter 10 in the IA-32 Intel Architecture Software Developer’s Manual, Volume 1.

Because the WC protocol uses a weakly-ordered memory consistency model, a fencing operation implemented with the SFENCE or MFENCE instruction should be used in conjunction with MOVNTPS instructions if multiple processors might use different memory types to read/write the destination memory locations.

Note: VEX.vvvv and EVEX.vvvv are reserved and must be 1111b otherwise instructions will #​​​UD.

## Operation

### VMOVNTPS (EVEX Encoded Versions)

```
VL = 128, 256, 512
DEST[VL-1:0] := SRC[VL-1:0]
DEST[MAXVL-1:VL] := 0

```

### MOVNTPS

```
DEST := SRC

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
VMOVNTPS void _mm512_stream_ps(float * p, __m512d a);

```

```
MOVNTPS void _mm_stream_ps (float * p, __m128d a);

```

```
VMOVNTPS void _mm256_stream_ps (float * p, __m256 a);

```

## SIMD Floating-Point Exceptions

None.

## Other Exceptions

Non-EVEX-encoded instruction, see Exceptions Type1.SSE in Table 2-18, “Type 1 Class Exception Conditions.”

EVEX-encoded instruction, see Table 2-45, “Type E1NF Class Exception Conditions.”

Additionally:

| #​​​UD | If VEX.vvvv != 1111B or EVEX.vvvv != 1111B. |
| ------ | ------------------------------------------- |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
