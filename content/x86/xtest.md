# XTEST

**Test if in Transactional Execution**

| Opcode/Instruction | Op/En | 64/32bit Mode Support | CPUID Feature Flag | Description                                  |
| ------------------ | ----- | --------------------- | ------------------ | -------------------------------------------- |
| NP 0F 01 D6 XTEST  | ZO    | V/V                   | HLE or RTM         | Test if executing in a transactional region. |

## Instruction Operand Encoding

| Op/En | Operand 1 | Operand2 | Operand3 | Operand4 |
| ----- | --------- | -------- | -------- | -------- |
| ZO    | N/A       | N/A      | N/A      | N/A      |

## Description

The XTEST instruction queries the transactional execution status. If the instruction executes inside a transactionally executing RTM region or a transactionally executing HLE region, then the ZF flag is cleared, else it is set.

## Operation

### XTEST

```
IF (RTM_ACTIVE = 1 OR HLE_ACTIVE = 1)
    THEN
        ZF := 0
    ELSE
        ZF := 1
FI;

```

## Flags Affected

The ZF flag is cleared if the instruction is executed transactionally; otherwise it is set to 1. The CF, OF, SF, PF, and AF, flags are cleared.

## Intel C/C++ Compiler Intrinsic Equivalent

```
XTEST int _xtest( void );

```

## SIMD Floating-Point Exceptions

None.

## Other Exceptions

| #​​​UD                  | CPUID.(EAX=7, ECX=0):EBX.HLE[bit 4] = 0 and CPUID.(EAX=7, ECX=0):EBX.RTM[bit 11] = 0. |
| ----------------------- | ------------------------------------------------------------------------------------- |
| If LOCK prefix is used. |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
