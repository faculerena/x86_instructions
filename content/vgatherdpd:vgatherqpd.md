# VGATHERDPD/VGATHERQPD

**Gather Packed Double Precision Floating**

| Opcode/Instruction                                    | Op/En | 64/32 Bit Mode Support | CPUID Feature Flag | Description                                                                                                                                                                                    |
| ----------------------------------------------------- | ----- | ---------------------- | ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| VEX.128.66.0F38.W1 92 /r VGATHERDPD xmm1, vm32x, xmm2 | RMV   | V/V                    | AVX2               | Using dword indices specified in vm32x, gather double precision floating-point values from memory conditioned on mask specified by xmm2. Conditionally gathered elements are merged into xmm1. |
| VEX.128.66.0F38.W1 93 /r VGATHERQPD xmm1, vm64x, xmm2 | RMV   | V/V                    | AVX2               | Using qword indices specified in vm64x, gather double precision floating-point values from memory conditioned on mask specified by xmm2. Conditionally gathered elements are merged into xmm1. |
| VEX.256.66.0F38.W1 92 /r VGATHERDPD ymm1, vm32x, ymm2 | RMV   | V/V                    | AVX2               | Using dword indices specified in vm32x, gather double precision floating-point values from memory conditioned on mask specified by ymm2. Conditionally gathered elements are merged into ymm1. |
| VEX.256.66.0F38.W1 93 /r VGATHERQPD ymm1, vm64y, ymm2 | RMV   | V/V                    | AVX2               | Using qword indices specified in vm64y, gather double precision floating-point values from memory conditioned on mask specified by ymm2. Conditionally gathered elements are merged into ymm1. |

## Instruction Operand Encoding

| Op/En | Operand 1       | Operand 2                                        | Operand 3       | Operand 4 |
| ----- | --------------- | ------------------------------------------------ | --------------- | --------- |
| RMV   | ModRM:reg (r,w) | BaseReg (R): VSIB:base, VectorReg(R): VSIB:index | VEX.vvvv (r, w) | N/A       |

## Description

The instruction conditionally loads up to 2 or 4 double precision floating-point values from memory addresses specified by the memory operand (the second operand) and using qword indices. The memory operand uses the VSIB form of the SIB byte to specify a general purpose register operand as the common base, a vector register for an array of indices relative to the base and a constant scale factor.

The mask operand (the third operand) specifies the conditional load operation from each memory address and the corresponding update of each data element of the destination operand (the first operand). Conditionality is specified by the most significant bit of each data element of the mask register. If an element’s mask bit is not set, the corresponding element of the destination register is left unchanged. The width of data element in the destination register and mask register are identical. The entire mask register will be set to zero by this instruction unless the instruction causes an exception.

Using dword indices in the lower half of the mask register, the instruction conditionally loads up to 2 or 4 double precision floating-point values from the VSIB addressing memory operand, and updates the destination register.

This instruction can be suspended by an exception if at least one element is already gathered (i.e., if the exception is triggered by an element other than the rightmost one with its mask bit set). When this happens, the destination register and the mask operand are partially updated; those elements that have been gathered are placed into the destination register and have their mask bits set to zero. If any traps or interrupts are pending from already gathered elements, they will be delivered in lieu of the exception; in this case, EFLAG.RF is set to one so an instruction breakpoint is not re-triggered when the instruction is continued.

If the data size and index size are different, part of the destination register and part of the mask register do not correspond to any elements being gathered. This instruction sets those parts to zero. It may do this to one or both of those registers even if the instruction triggers an exception, and even if the instruction triggers the exception before gathering any elements.

VEX.128 version: The instruction will gather two double precision floating-point values. For dword indices, only the lower two indices in the vector index register are used.

VEX.256 version: The instruction will gather four double precision floating-point values. For dword indices, only the lower four indices in the vector index register are used.

Note that:

- If any pair of the index, mask, or destination registers are the same, this instruction results a #​​​UD fault.
- The values may be read from memory in any order. Memory ordering with other instructions follows the Intel-64 memory-ordering model.
- Faults are delivered in a right-to-left manner. That is, if a fault is triggered by an element and delivered, all elements closer to the LSB of the destination will be completed (and non-faulting). Individual elements closer to the MSB may or may not be completed. If a given element triggers multiple faults, they are delivered in the conventional order.
- Elements may be gathered in any order, but faults must be delivered in a right-to-left order; thus, elements to the left of a faulting one may be gathered before the fault is delivered. A given implementation of this instruction is repeatable - given the same input values and architectural state, the same set of elements to the left of the faulting one will be gathered.
- This instruction does not perform AC checks, and so will never deliver an AC fault.
- This instruction will cause a #​​​UD if the address size attribute is 16-bit.
- This instruction will cause a #​​​UD if the memory operand is encoded without the SIB byte.
- This instruction should not be used to access memory mapped I/O as the ordering of the individual loads it does is implementation specific, and some implementations may use loads larger than the data element size or load elements an indeterminate number of times.
- The scaled index may require more bits to represent than the address bits used by the processor (e.g., in 32-bit mode, if the scale is greater than one). In this case, the most significant bits beyond the number of address bits are ignored.

## Operation

```
DEST := SRC1;
BASE_ADDR: base register encoded in VSIB addressing;
VINDEX: the vector index register encoded by VSIB addressing;
SCALE: scale factor encoded by SIB:[7:6];
DISP: optional 1, 4 byte displacement;
MASK := SRC3;

```

### VGATHERDPD (VEX.128 version)

```
MASK[MAXVL-1:128] := 0;
FOR j := 0 to 1
    i := j * 64;
    IF MASK[63+i] THEN
        MASK[i +63:i] := FFFFFFFF_FFFFFFFFH; // extend from most significant bit
    ELSE
        MASK[i +63:i] := 0;
    FI;
ENDFOR
FOR j := 0 to 1
    k := j * 32;
    i := j * 64;
    DATA_ADDR := BASE_ADDR + (SignExtend(VINDEX[k+31:k])*SCALE + DISP;
    IF MASK[63+i] THEN
        DEST[i +63:i] := FETCH_64BITS(DATA_ADDR); // a fault exits the instruction
    FI;
    MASK[i +63: i] := 0;
ENDFOR
DEST[MAXVL-1:128] := 0;

```

### VGATHERQPD (VEX.128 version)

```
MASK[MAXVL-1:128] := 0;
FOR j := 0 to 1
    i := j * 64;
    IF MASK[63+i] THEN
        MASK[i +63:i] := FFFFFFFF_FFFFFFFFH; // extend from most significant bit
    ELSE
        MASK[i +63:i] := 0;
    FI;
ENDFOR
FOR j := 0 to 1
    i := j * 64;
    DATA_ADDR := BASE_ADDR + (SignExtend(VINDEX1[i+63:i])*SCALE + DISP;
    IF MASK[63+i] THEN
        DEST[i +63:i] := FETCH_64BITS(DATA_ADDR); // a fault exits this instruction
    FI;
    MASK[i +63: i] := 0;
ENDFOR
DEST[MAXVL-1:128] := 0;

```

### VGATHERQPD (VEX.256 version)

```
MASK[MAXVL-1:256] := 0;
FOR j := 0 to 3
    i := j * 64;
    IF MASK[63+i] THEN
        MASK[i +63:i] := FFFFFFFF_FFFFFFFFH; // extend from most significant bit
    ELSE
        MASK[i +63:i] := 0;
    FI;
ENDFOR
FOR j := 0 to 3
    i := j * 64;
    DATA_ADDR := BASE_ADDR + (SignExtend(VINDEX1[i+63:i])*SCALE + DISP;
    IF MASK[63+i] THEN
        DEST[i +63:i] := FETCH_64BITS(DATA_ADDR); // a fault exits the instruction
    FI;
    MASK[i +63: i] := 0;
ENDFOR
DEST[MAXVL-1:256] := 0;

```

### VGATHERDPD (VEX.256 version)

```
MASK[MAXVL-1:256] := 0;
FOR j := 0 to 3
    i := j * 64;
    IF MASK[63+i] THEN
        MASK[i +63:i] := FFFFFFFF_FFFFFFFFH; // extend from most significant bit
    ELSE
        MASK[i +63:i] := 0;
    FI;
ENDFOR
FOR j := 0 to 3
    k := j * 32;
    i := j * 64;
    DATA_ADDR := BASE_ADDR + (SignExtend(VINDEX1[k+31:k])*SCALE + DISP;
    IF MASK[63+i] THEN
        DEST[i +63:i] := FETCH_64BITS(DATA_ADDR); // a fault exits the instruction
    FI;
    MASK[i +63:i] := 0;
ENDFOR
DEST[MAXVL-1:256] := 0;

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
VGATHERDPD: __m128d _mm_i32gather_pd (double const * base, __m128i index, const int scale);

```

```
VGATHERDPD: __m128d _mm_mask_i32gather_pd (__m128d src, double const * base, __m128i index, __m128d mask, const int scale);

```

```
VGATHERDPD: __m256d _mm256_i32gather_pd (double const * base, __m128i index, const int scale);

```

```
VGATHERDPD: __m256d _mm256_mask_i32gather_pd (__m256d src, double const * base, __m128i index, __m256d mask, const int scale);

```

```
VGATHERQPD: __m128d _mm_i64gather_pd (double const * base, __m128i index, const int scale);

```

```
VGATHERQPD: __m128d _mm_mask_i64gather_pd (__m128d src, double const * base, __m128i index, __m128d mask, const int scale);

```

```
VGATHERQPD: __m256d _mm256_i64gather_pd (double const * base, __m256i index, const int scale);

```

```
VGATHERQPD: __m256d _mm256_mask_i64gather_pd (__m256d src, double const * base, __m256i index, __m256d mask, const int scale);

```

## SIMD Floating-Point Exceptions

None.

## Other Exceptions

See Table 2-27, “Type 12 Class Exception Conditions.”

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
