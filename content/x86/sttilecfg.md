#STTILECFG
**Store Tile Configuration**

| Opcode/Instruction                                 | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                       |
| -------------------------------------------------- | ----- | ---------------------- | ------------------ | --------------------------------- |
| VEX.128.66.0F38.W0 49 !(11):000:bbb STTILECFG m512 | A     | V/N.E.                 | AMX-TILE           | Store tile configuration in m512. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1     | Operand 2 | Operand 3 | Operand 4 |
| ----- | ----- | ------------- | --------- | --------- | --------- |
| A     | N/A   | ModRM:r/m (w) | N/A       | N/A       | N/A       |

## Description

The STTILECFG instruction takes a pointer to a 64-byte memory location (described in [Table 3-10](/x86/cpuid#tbl-3-10) in the “LDTILECFG—Load Tile Configuration” entry) that will, after successful execution of this instruction, contain the description of the tiles that were configured. In order to configure tiles, the AMX-TILE bit in CPUID must be set and the operating system has to have enabled the tiles architecture.

If the tiles are not configured, then STTILECFG stores 64B of zeros to the indicated memory location.

Any attempt to execute the STTILECFG instruction inside an Intel TSX transaction will result in a transaction abort.

## Operation

### STTILECFG mem

```
if TILES_CONFIGURED == 0:
    //write 64 bytes of zeros at mem pointer
    buf[0..63] := 0
    write_memory(mem, 64, buf)
else:
    buf.byte[0] := tilecfg.palette_id
    buf.byte[1] := tilecfg.start_row
    buf.byte[2..15] := 0
    p := 16
    for n in 0 ... palette_table[tilecfg.palette_id].max_names-1:
        buf.word[p/2] := tilecfg.t[n].colsb
        p := p + 2
    if p < 47:
        buf.byte[p..47] := 0
    p := 48
    for n in 0 ... palette_table[tilecfg.palette_id].max_names-1:
        buf.byte[p++] := tilecfg.t[n].rows
    if p < 63:
        buf.byte[p..63] := 0
    write_memory(mem, 64, buf)

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
STTILECFGvoid _tile_storeconfig(void *);

```

## Flags Affected

None.

## Exceptions

AMX-E2; see Section 2.10, “Intel® AMX Instruction Exception Classes,” for details.

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
