#LDTILECFG
**Load Tile Configuration**

| Opcode/Instruction                                 | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                   |
| -------------------------------------------------- | ----- | ---------------------- | ------------------ | --------------------------------------------- |
| VEX.128.NP.0F38.W0 49 !(11):000:bbb LDTILECFG m512 | A     | V/N.E.                 | AMX-TILE           | Load tile configuration as specified in m512. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1     | Operand 2 | Operand 3 | Operand 4 |
| ----- | ----- | ------------- | --------- | --------- | --------- |
| A     | N/A   | ModRM:r/m (r) | N/A       | N/A       | N/A       |

### Description

The LDTILECFG instruction takes an operand containing a pointer to a 64-byte memory location containing the description of the tiles to be supported. In order to configure the tiles, the AMX-TILE bit in CPUID must be set and the operating system has to have enabled the tiles architecture.

The memory area contains the palette and describes how many tiles are being used and defines each tile in terms of rows and column bytes. Requests must be compatible with the restrictions provided by CPUID; see [Table 3-10](/x86/cpuid#tbl-3-10) below.

| Byte(s) | Field Name             | Description                                                                  |
| ------- | ---------------------- | ---------------------------------------------------------------------------- |
| 0       | palette                | Palette selects the supported configuration of the tiles that will be used.  |
| 1       | start_row              | start_row is used for storing the restart values for interrupted operations. |
| 2-15    | reserved, must be zero |                                                                              |
| 16-17   | tile0.colsb            | Tile 0 bytes per row.                                                        |
| 18-19   | tile1.colsb            | Tile 1 bytes per row.                                                        |
| 20-21   | tile2.colsb            | Tile 2 bytes per row.                                                        |
| ...     | (sequence continues)   |                                                                              |
| 30-31   | tile7.colsb            | Tile 7 bytes per row.                                                        |
| 32-47   | reserved, must be zero |                                                                              |
| 48      | tile0.rows             | Tile 0 rows.                                                                 |
| 49      | tile1.rows             | Tile 1 rows.                                                                 |
| 50      | tile2.rows             | Tile 2 rows.                                                                 |
| ...     | (sequence continues)   |                                                                              |
| 55      | tile7.rows             | Tile 7 rows.                                                                 |
| 56-63   | reserved, must be zero |                                                                              |

[Table 3-10](/x86/cpuid#tbl-3-10). Memory Area Layout

If a tile row and column pair is not used to specify tile parameters, they must have the value zero. All enabled tiles (based on the palette) must be configured. Specifying tile parameters for more tiles than the implementation limit or the palette limit results in a #​​​​GP fault.

If the palette_id is zero, that signifies the INIT state for both TILECFG and TILEDATA. Tiles are zeroed in the INIT state. The only legal non-INIT value for palette_id is 1.

Any attempt to execute the LDTILECFG instruction inside an Intel TSX transaction will result in a transaction abort.

### Operation

#### LDTILECFG mem

```
error := False
buf := read_memory(mem, 64)
temp_tilecfg.palette_id := buf.byte[0]
if temp_tilecfg.palette_id > max_palette:
    error := True
if not xcr0_supports_palette(temp_tilecfg.palette_id):
    error := True
if temp_tilecfg.palette_id !=0:
    temp_tilecfg.start_row := buf.byte[1]
    if buf.byte[2..15] is nonzero:
        error := True
    p := 16
    # configure columns
    for n in 0 ... palette_table[temp_tilecfg.palette_id].max_names-1:
        temp_tilecfg.t[n].colsb:= buf.word[p/2]
        p := p + 2
        if temp_tilecfg.t[n].colsb > palette_table[temp_tilecfg.palette_id].bytes_per_row:
            error := True
    if nonzero(buf[p...47]):
        error := True
    # configure rows
    p := 48
    for n in 0 ... palette_table[temp_tilecfg.palette_id].max_names-1:
        temp_tilecfg.t[n].rows:= buf.byte[p]
        if temp_tilecfg.t[n].rows > palette_table[temp_tilecfg.palette_id].max_rows:
            error := True
        p := p + 1
    if nonzero(buf[p...63]):
        error := True
    # validate each tile's row & col configs are reasonable and enable the valid tiles
    for n in 0 ... palette_table[temp_tilecfg.palette_id].max_names-1:
        if temp_tilecfg.t[n].rows !=0 and temp_tilecfg.t[n].colsb != 0:
            temp_tilecfg.t[n].valid := 1
        elif temp_tilecfg.t[n].rows == 0 and temp_tilecfg.t[n].colsb == 0:
            temp_tilecfg.t[n].valid := 0
        else:
            error := True// one of rows or colsbwas 0 but not both.
if error:
    #​​​​GP
elif temp_tilecfg.palette_id == 0:
    TILES_CONFIGURED := 0// init state
    tilecfg := 0// equivalent to 64B of zeros
    zero_all_tile_data()
else:
    tilecfg := temp_tilecfg
    zero_all_tile_data()
    TILES_CONFIGURED := 1

```

### Intel C/C++ Compiler Intrinsic Equivalent

```
LDTILECFG void _tile_loadconfig(const void *);

```

### Flags Affected

None.

### Exceptions

AMX-E1; see Section 2.10, “Intel® AMX Instruction Exception Classes,” for details.

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
