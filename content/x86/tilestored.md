#TILESTORED
**Store Tile**

| Opcode/Instruction                                          | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                  |
| ----------------------------------------------------------- | ----- | ---------------------- | ------------------ | -------------------------------------------- |
| VEX.128.F3.0F38.W0 4B !(11):rrr:100 TILESTORED sibmem, tmm1 | A     | V/N.E.                 | AMX-TILE           | Store a tile in sibmem as specified in tmm1. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1     | Operand 2     | Operand 3 | Operand 4 |
| ----- | ----- | ------------- | ------------- | --------- | --------- |
| A     | N/A   | ModRM:r/m (w) | ModRM:reg (r) | N/A       | N/A       |

## Description

This instruction is required to use SIB addressing. The index register serves as a stride indicator. If the SIB encoding omits an index register, the value zero is assumed for the content of the index register.

This instruction stores a tile source of rows and columns as specified by the tile configuration.

The TILECFG.start_row in the TILECFG data should be initialized to '0' in order to store the entire tile and are set to zero on successful completion of the TILESTORED instruction. TILESTORED is a restartable instruction and the TILECFG.start_row will be non-zero when restartable events occur during the instruction execution.

Only memory operands are supported and they can only be accessed using a SIB addressing mode, similar to the V[P]GATHER\*/V[P]SCATTER\* instructions.

Any attempt to execute the TILESTORED instruction inside an Intel TSX transaction will result in a transaction abort.

## Operation

```
TILESTORED tsib, tsrc
start := tilecfg.start_row
membegin := tsib.base + displacement
// if no index register in the SIB encoding, the value zero is used.
stride := tsib.index << tsib.scale
while start < tdest.rows:
    memptr := membegin + start * stride
    write_memory(memptr, tsrc.colsb, tsrc.row[start])
    start := start + 1
zero_tilecfg_start()
// In the case of a memory fault in the middle of an instruction, the tilecfg.start_row := start

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
TILESTORED void _tile_stored(__tile src, void *base, int stride);

```

## Flags Affected

None.

## Exceptions

AMX-E3; see Section 2.10, “Intel® AMX Instruction Exception Classes,” for details.

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
