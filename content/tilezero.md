# TILEZERO

**Zero Tile**

| Opcode/Instruction                             | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                |
| ---------------------------------------------- | ----- | ---------------------- | ------------------ | -------------------------- |
| VEX.128.F2.0F38.W0 49 11:rrr:000 TILEZERO tmm1 | A     | V/N.E.                 | AMX-TILE           | Zero the destination tile. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1     | Operand 2 | Operand 3 | Operand 4 |
| ----- | ----- | ------------- | --------- | --------- | --------- |
| A     | N/A   | ModRM:reg (w) | N/A       | N/A       | N/A       |

## Description

This instruction zeroes the destination tile.

Any attempt to execute the TILEZERO instruction inside an Intel TSX transaction will result in a transaction abort.

## Operation

```
TILEZERO tdest
nbytes := palette_table[palette_id].bytes_per_row
for i in 0 ... palette_table[palette_id].max_rows-1:
    for j in 0 ... nbytes-1:
        tdest.row[i].byte[j] := 0
zero_tilecfg_start()

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
TILEZERO void _tile_zero(__tile dst);

```

## Flags Affected

None.

## Exceptions

AMX-E5; see Section 2.10, “Intel® AMX Instruction Exception Classes,” for details.

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
