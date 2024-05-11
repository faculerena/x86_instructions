# LOOP/LOOPcc

**Loop According to ECX Counter**

| Opcode | Instruction | Op/En | 64-Bit Mode | Compat/Leg Mode | Description                                          |
| ------ | ----------- | ----- | ----------- | --------------- | ---------------------------------------------------- |
| E2 cb  | LOOP rel8   | D     | Valid       | Valid           | Decrement count; jump short if count ≠ 0.            |
| E1 cb  | LOOPE rel8  | D     | Valid       | Valid           | Decrement count; jump short if count ≠ 0 and ZF = 1. |
| E0 cb  | LOOPNE rel8 | D     | Valid       | Valid           | Decrement count; jump short if count ≠ 0 and ZF = 0. |

## Instruction Operand Encoding

| Op/En Operand 1 Operand 2 Operand 3 |     |     |     | Operand 4 |
| ----------------------------------- | --- | --- | --- | --------- |
| D Offset N/A N/A                    |     |     |     | N/A       |

## Description

Performs a loop operation using the RCX, ECX or CX register as a counter (depending on whether address size is 64 bits, 32 bits, or 16 bits). Note that the LOOP instruction ignores REX.W; but 64-bit address size can be over-ridden using a 67H prefix.

Each time the LOOP instruction is executed, the count register is decremented, then checked for 0. If the count is 0, the loop is terminated and program execution continues with the instruction following the LOOP instruction. If the count is not zero, a near jump is performed to the destination (target) operand, which is presumably the instruction at the beginning of the loop.

The target instruction is specified with a relative offset (a signed offset relative to the current value of the instruction pointer in the IP/EIP/RIP register). This offset is generally specified as a label in assembly code, but at the machine code level, it is encoded as a signed, 8-bit immediate value, which is added to the instruction pointer. Offsets of –128 to +127 are allowed with this instruction.

Some forms of the loop instruction (LOOP*cc*) also accept the ZF flag as a condition for terminating the loop before the count reaches zero. With these forms of the instruction, a condition code (_cc_) is associated with each instruction to indicate the condition being tested for. Here, the LOOP*cc* instruction itself does not affect the state of the ZF flag; the ZF flag is changed by other instructions in the loop.

## Operation

```
IF (AddressSize = 32)
    THEN Count is ECX;
ELSE IF (AddressSize = 64)
    Count is RCX;
ELSE Count is CX;
FI;
Count := Count – 1;
IF Instruction is not LOOP
    THEN
        IF (Instruction := LOOPE) or (Instruction := LOOPZ)
            THEN IF (ZF = 1) and (Count ≠ 0)
                    THEN BranchCond := 1;
                    ELSE BranchCond := 0;
                FI;
            ELSE (Instruction = LOOPNE) or (Instruction = LOOPNZ)
                IF (ZF = 0 ) and (Count ≠ 0)
                    THEN BranchCond := 1;
                    ELSE BranchCond := 0;
        FI;
    ELSE (* Instruction = LOOP *)
        IF (Count ≠ 0)
            THEN BranchCond := 1;
            ELSE BranchCond := 0;
        FI;
FI;
IF BranchCond = 1
    THEN
        IF in 64-bit mode (* OperandSize = 64 *)
            THEN
                tempRIP := RIP + SignExtend(DEST);
                IF tempRIP is not canonical
                    THEN #​​​​GP(0);
                ELSE RIP := tempRIP;
                FI;
            ELSE
                tempEIP := EIP SignExtend(DEST);
                IF OperandSize 16
                    THEN tempEIP := tempEIP AND 0000FFFFH;
                FI;
                IF tempEIP is not within code segment limit
                    THEN #​​​​GP(0);
                    ELSE EIP := tempEIP;
                FI;
        FI;
    ELSE
        Terminate loop and continue program execution at (R/E)IP;
FI;

```

## Flags Affected

None.

## Protected Mode Exceptions

| \#​​​​GP(0) | If the offset being jumped to is beyond the limits of the CS segment. |
| ----------- | --------------------------------------------------------------------- |
| #​​​UD      | If the LOCK prefix is used.                                           |

## Real-Address Mode Exceptions

| \#​​​​GP | If the offset being jumped to is beyond the limits of the CS segment or is outside of the effective address space from 0 to FFFFH. This condition can occur if a 32-bit address size override prefix is used. |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| #​​​UD   | If the LOCK prefix is used.                                                                                                                                                                                   |

## Virtual-8086 Mode Exceptions

Same exceptions as in real address mode.

## Compatibility Mode Exceptions

Same exceptions as in protected mode.

## 64-Bit Mode Exceptions

| \#​​​​GP(0) | If the offset being jumped to is in a non-canonical form. |
| ----------- | --------------------------------------------------------- |
| #​​​UD      | If the LOCK prefix is used.                               |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
