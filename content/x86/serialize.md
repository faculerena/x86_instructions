# SERIALIZE**Serialize Instruction Execution**

| Opcode/Instruction    | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                |
| --------------------- | ----- | ---------------------- | ------------------ | ------------------------------------------ |
| NP 0F 01 E8 SERIALIZE | ZO    | V/V                    | SERIALIZE          | Serialize instruction fetch and execution. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1 | Operand 2 | Operand 3 | Operand 4 |
| ----- | ----- | --------- | --------- | --------- | --------- |
| ZO    | N/A   | N/A       | N/A       | N/A       | N/A       |

## Description

Serializes instruction execution. Before the next instruction is fetched and executed, the SERIALIZE instruction ensures that all modifications to flags, registers, and memory by previous instructions are completed, draining all buffered writes to memory. This instruction is also a serializing instruction as defined in the section “Serializing Instructions” in Chapter 9 of the Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 3A.

SERIALIZE does not modify registers, arithmetic flags, or memory.

## Operation

```
Wait_On_Fetch_And_Execution_Of_Next_Instruction_Until(preceding_instructions_complete_and_preceding_stores_globally_visible);

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
SERIALIZE void _serialize(void);

```

## SIMD Floating-Point Exceptions

None.

## Other Exceptions

| #​​​UD                                     | If the LOCK prefix is used. |
| ------------------------------------------ | --------------------------- |
| If CPUID.07H.0H:EDX.SERIALIZE[bit 14] = 0. |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
