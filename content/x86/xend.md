# XEND**Transactional End**

| Opcode/Instruction | Op/En | 64/32bit Mode Support | CPUID Feature Flag | Description                              |
| ------------------ | ----- | --------------------- | ------------------ | ---------------------------------------- |
| NP 0F 01 D5 XEND   | A     | V/V                   | RTM                | Specifies the end of an RTM code region. |

## Instruction Operand Encoding

| Op/En | Operand 1 | Operand2 | Operand3 | Operand4 |
| ----- | --------- | -------- | -------- | -------- |
| A     | N/A       | N/A      | N/A      | N/A      |

## Description

The instruction marks the end of an RTM code region. If this corresponds to the outermost scope (that is, including this XEND instruction, the number of XBEGIN instructions is the same as number of XEND instructions), the logical processor will attempt to commit the logical processor state atomically. If the commit fails, the logical processor will rollback all architectural register and memory updates performed during the RTM execution. The logical processor will resume execution at the fallback address computed from the outermost XBEGIN instruction. The EAX register is updated to reflect RTM abort information.

Execution of XEND outside a transactional region causes a general-protection exception (#​​​​GP). Execution of XEND while in a suspend read address tracking region causes a transactional abort.

## Operation

### XEND

```
IF (RTM_ACTIVE = 0) THEN
    SIGNAL #​​​​GP
ELSE
    IF SUSLDTRK_ACTIVE = 1
        THEN GOTO RTM_ABORT_PROCESSING;
    FI;
    RTM_NEST_COUNT--
    IF (RTM_NEST_COUNT = 0) THEN
        Try to commit transaction
        IF fail to commit transactional execution
            THEN
                GOTO RTM_ABORT_PROCESSING;
            ELSE (* commit success *)
                RTM_ACTIVE := 0
        FI;
    FI;
FI;
(* For any RTM abort condition encountered during RTM execution *)
RTM_ABORT_PROCESSING:
    Restore architectural register state
    Discard memory updates performed in transaction
    Update EAX with status
    RTM_NEST_COUNT := 0
    RTM_ACTIVE := 0
    SUSLDTRK_ACTIVE := 0
    IF 64-bit Mode
        THEN
            RIP := fallbackRIP
        ELSE
            EIP := fallbackEIP
    FI;
END

```

## Flags Affected

None.

## Intel C/C++ Compiler Intrinsic Equivalent

```
XEND void _xend( void );

```

## SIMD Floating-Point Exceptions

None.

## Other Exceptions

| #​​​UD                  | CPUID.(EAX=7, ECX=0):EBX.RTM[bit 11] = 0. |
| ----------------------- | ----------------------------------------- |
| If LOCK prefix is used. |
| \#​​​​GP(0)             | If RTM_ACTIVE = 0.                        |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
