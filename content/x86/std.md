# STD**Set Direction Flag**

| Opcode | Instruction | Op/En | 64-bit Mode | Compat/Leg Mode | Description  |
| ------ | ----------- | ----- | ----------- | --------------- | ------------ |
| FD     | STD         | ZO    | Valid       | Valid           | Set DF flag. |

## Instruction Operand Encoding

| Op/En | Operand 1 | Operand 2 | Operand 3 | Operand 4 |
| ----- | --------- | --------- | --------- | --------- |
| ZO    | N/A       | N/A       | N/A       | N/A       |

## Description

Sets the DF flag in the EFLAGS register. When the DF flag is set to 1, string operations decrement the index registers (ESI and/or EDI). Operation is the same in all modes.

## Operation

```
DF := 1;

```

## Flags Affected

The DF flag is set. The CF, OF, ZF, SF, AF, and PF flags are unaffected.

## Exceptions (All Operating Modes)

#​​​UD If the LOCK prefix is used.

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
