#TILELOADD/TILELOADDT1
**Load Tile**

| Opcode/Instruction                                           | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                                   |
| ------------------------------------------------------------ | ----- | ---------------------- | ------------------ | --------------------------------------------------------------------------------------------- |
| VEX.128.F2.0F38.W0 4B !(11):rrr:100 TILELOADD tmm1, sibmem   | A     | V/N.E.                 | AMX-TILE           | Load data into tmm1 as specified by information in sibmem.                                    |
| VEX.128.66.0F38.W0 4B !(11):rrr:100 TILELOADDT1 tmm1, sibmem | A     | V/N.E.                 | AMX-TILE           | Load data into tmm1 as specified by information in sibmem with hint to optimize data caching. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1     | Operand 2     | Operand 3 | Operand 4 |
| ----- | ----- | ------------- | ------------- | --------- | --------- |
| A     | N/A   | ModRM:reg (w) | ModRM:r/m (r) | N/A       | N/A       |

## Description

This instruction is required to use SIB addressing. The index register serves as a stride indicator. If the SIB encoding omits an index register, the value zero is assumed for the content of the index register.

This instruction loads a tile destination with rows and columns as specified by the tile configuration. The “T1” version provides a hint to the implementation that the data would be reused but does not need to be resident in the nearest cache levels.

The TILECFG.start_row in the TILECFG data should be initialized to '0' in order to load the entire tile and is set to zero on successful completion of the TILELOADD instruction. TILELOADD is a restartable instruction and the TILECFG.start_row will be non-zero when restartable events occur during the instruction execution.

Only memory operands are supported and they can only be accessed using a SIB addressing mode, similar to the V[P]GATHER\*/V[P]SCATTER\* instructions.

Any attempt to execute the TILELOADD/TILELOADDT1 instructions inside an Intel TSX transaction will result in a transaction abort.

## Operation

```
TILELOADD[,T1] tdest, tsib
start := tilecfg.start_row
zero_upper_rows(tdest,start)
membegin := tsib.base + displacement
// if no index register in the SIB encoding, the value zero is used.
stride := tsib.index << tsib.scale
nbytes := tdest.colsb
while start < tdest.rows:
    memptr := membegin + start * stride
    write_row_and_zero(tdest, start, read_memory(memptr, nbytes), nbytes)
    start := start + 1
zero_tilecfg_start()
// In the case of a memory fault in the middle of an instruction, the tilecfg.start_row := start

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
TILELOADD void _tile_loadd(__tile dst, const void *base, int stride);

```

```
TILELOADDT1 void _tile_stream_loadd(__tile dst, const void *base, int stride);

```

## Flags Affected

None.

## Exceptions

AMX-E3; see Section 2.10, “Intel® AMX Instruction Exception Classes,” for details.

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
