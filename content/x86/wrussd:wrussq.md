#WRUSSD/WRUSSQ
**Write to User Shadow Stack**

| Opcode/Instruction                              | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                    |
| ----------------------------------------------- | ----- | ---------------------- | ------------------ | ------------------------------ |
| 66 0F 38 F5 !(11):rrr:bbb WRUSSD m32, r32       | MR    | V/V                    | CET_SS             | Write 4 bytes to shadow stack. |
| 66 REX.W 0F 38 F5 !(11):rrr:bbb WRUSSQ m64, r64 | MR    | V/N.E.                 | CET_SS             | Write 8 bytes to shadow stack. |

## Instruction Operand Encoding

| Op/En | Operand 1     | Operand 2     | Operand 3 | Operand 4 |
| ----- | ------------- | ------------- | --------- | --------- |
| MR    | ModRM:r/m (w) | ModRM:reg (r) | N/A       | N/A       |

## Description

Writes bytes in register source to a user shadow stack page. The WRUSS instruction can be executed only if CPL = 0, however the processor treats its shadow-stack accesses as user accesses.

## Operation

```
IF CR4.CET = 0
    THEN #​​​UD; FI;
IF CPL > 0
    THEN #​​​​GP(0); FI;
DEST_LA = Linear_Address(mem operand)
IF (operand size is 64 bit)
    THEN
        (* Destination not 8B aligned *)
        IF DEST_LA[2:0]
            THEN GP(0); FI;
        Shadow_stack_store 8 bytes of SRC to DEST_LA as user-mode access;
    ELSE
        (* Destination not 4B aligned *)
        IF DEST_LA[1:0]
            THEN GP(0); FI;
        Shadow_stack_store 4 bytes of SRC[31:0] to DEST_LA as user-mode access;
FI;

```

## Flags Affected

None.

## C/C++ Compiler Intrinsic Equivalent

```
WRUSSD void _wrussd(__int32, void *);

```

```
WRUSSQ void _wrussq(__int64, void *);

```

## Protected Mode Exceptions

| #​​​UD                                                                                              | If the LOCK prefix is used.                                                               |
| --------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| If CR4.CET = 0.                                                                                     |
| \#​​​​GP(0)                                                                                         | If a memory operand effective address is outside the CS, DS, ES, FS, or GS segment limit. |
| If destination is located in a non-writeable segment.                                               |
| If the DS, ES, FS, or GS register is used to access memory and it contains a NULL segment selector. |
| If linear address of destination is not 4 byte aligned.                                             |
| If CPL is not 0.                                                                                    |
| \#​​​​​SS(0)                                                                                        | If a memory operand effective address is outside the SS segment limit.                    |
| \#​PF(fault-code)                                                                                   | If destination is not a user shadow stack.                                                |
| Other terminal and non-terminal faults.                                                             |

## Real-Address Mode Exceptions

| #​​​UD | The WRUSS instruction is not recognized in real-address mode. |
| ------ | ------------------------------------------------------------- |

## Virtual-8086 Mode Exceptions

| #​​​UD | The WRUSS instruction is not recognized in virtual-8086 mode. |
| ------ | ------------------------------------------------------------- |

## Compatibility Mode Exceptions

| #​​​UD                                                  | If the LOCK prefix is used.                                                |
| ------------------------------------------------------- | -------------------------------------------------------------------------- |
| If CR4.CET = 0.                                         |
| \#​​​​GP(0)                                             | If a memory address is in a non-canonical form.                            |
| If linear address of destination is not 4 byte aligned. |
| If CPL is not 0.                                        |
| \#​​​​​SS(0)                                            | If a memory address referencing the SS segment is in a non-canonical form. |
| \#​PF(fault-code)                                       | If destination is not a user shadow stack.                                 |
| Other terminal and non-terminal faults.                 |

## 64-Bit Mode Exceptions

| #​​​UD                                                  | If the LOCK prefix is used.                     |
| ------------------------------------------------------- | ----------------------------------------------- |
| If CR4.CET = 0.                                         |
| \#​​​​GP(0)                                             | If a memory address is in a non-canonical form. |
| If linear address of destination is not 4 byte aligned. |
| If CPL is not 0.                                        |
| \#​PF(fault-code)                                       | If destination is not a user shadow stack.      |
| Other terminal and non-terminal faults.                 |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
