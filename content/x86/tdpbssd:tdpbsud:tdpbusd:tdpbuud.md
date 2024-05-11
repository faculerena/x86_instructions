#TDPBSSD/TDPBSUD/TDPBUSD/TDPBUUD
**Dot Product of Signed**

| Opcode/Instruction                                        | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                                                                                     |
| --------------------------------------------------------- | ----- | ---------------------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------- |
| VEX.128.F2.0F38.W0 5E 11:rrr:bbb TDPBSSD tmm1, tmm2, tmm3 | A     | V/N.E.                 | AMX-INT8           | Matrix multiply signed byte elements from tmm2 by signed byte elements from tmm3 and accumulate the dword elements in tmm1.     |
| VEX.128.F3.0F38.W0 5E 11:rrr:bbb TDPBSUD tmm1, tmm2, tmm3 | A     | V/N.E.                 | AMX-INT8           | Matrix multiply signed byte elements from tmm2 by unsigned byte elements from tmm3 and accumulate the dword elements in tmm1.   |
| VEX.128.66.0F38.W0 5E 11:rrr:bbb TDPBUSD tmm1, tmm2, tmm3 | A     | V/N.E.                 | AMX-INT8           | Matrix multiply unsigned byte elements from tmm2 by signed byte elements from tmm3 and accumulate the dword elements in tmm1.   |
| VEX.128.NP.0F38.W0 5E 11:rrr:bbb TDPBUUD tmm1, tmm2, tmm3 | A     | V/N.E.                 | AMX-INT8           | Matrix multiply unsigned byte elements from tmm2 by unsigned byte elements from tmm3 and accumulate the dword elements in tmm1. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1        | Operand 2     | Operand 3    | Operand 4 |
| ----- | ----- | ---------------- | ------------- | ------------ | --------- |
| A     | N/A   | ModRM:reg (r, w) | ModRM:r/m (r) | VEX.vvvv (r) | N/A       |

## Description

For each possible combination of (row of tmm2, column of tmm3), the instruction performs a set of SIMD dot-products on all corresponding four byte elements, one from tmm2 and one from tmm3, adds the results of those dot-products, and then accumulates the result into the corresponding row and column of tmm1. Each dword in input tiles tmm2 and tmm3 is interpreted as four byte elements. These may be signed or unsigned. Each letter in the two-letter pattern SU, US, SS, UU indicates the signed/unsigned nature of the values in tmm2 and tmm3, respectively.

Any attempt to execute the TDPBSSD/TDPBSUD/TDPBUSD/TDPBUUD instructions inside an Intel TSX transaction will result in a transaction abort.

## Operation

```
define DPBD(c,x,y):// arguments are dwords
    if *x operand is signed*:
        extend_src1 := SIGN_EXTEND
    else:
        extend_src1 := ZERO_EXTEND
    if *y operand is signed*:
        extend_src2 := SIGN_EXTEND
    else:
        extend_src2 := ZERO_EXTEND
    p0dword := extend_src1(x.byte[0]) * extend_src2(y.byte[0])
    p1dword := extend_src1(x.byte[1]) * extend_src2(y.byte[1])
    p2dword := extend_src1(x.byte[2]) * extend_src2(y.byte[2])
    p3dword := extend_src1(x.byte[3]) * extend_src2(y.byte[3])
    c := c + p0dword + p1dword + p2dword + p3dword

```

### TDPBSSD, TDPBSUD, TDPBUSD, TDPBUUD tsrcdest, tsrc1, tsrc2 (Register Only Version)

```
// C = m x n (tsrcdest), A = m x k (tsrc1), B = k x n (tsrc2)
tsrc1_elements_per_row := tsrc1.colsb / 4
tsrc2_elements_per_row := tsrc2.colsb / 4
tsrcdest_elements_per_row := tsrcdest.colsb / 4
for m in 0 ... tsrcdest.rows-1:
    tmp := tsrcdest.row[m]
    for k in 0 ... tsrc1_elements_per_row-1:
        for n in 0 ... tsrcdest_elements_per_row-1:
            DPBD( tmp.dword[n], tsrc1.row[m].dword[k], tsrc2.row[k].dword[n] )
    write_row_and_zero(tsrcdest, m, tmp, tsrcdest.colsb)
zero_upper_rows(tsrcdest, tsrcdest.rows)
zero_tilecfg_start()

```

## Intel C/C++ Compiler Intrinsic Equivalent

```
TDPBSSD void _tile_dpbssd(__tile dst, __tile src1, __tile src2);

```

```
TDPBSUD void _tile_dpbsud(__tile dst, __tile src1, __tile src2);

```

```
TDPBUSD void _tile_dpbusd(__tile dst, __tile src1, __tile src2);

```

```
TDPBUUD void _tile_dpbuud(__tile dst, __tile src1, __tile src2);

```

## Flags Affected

None.

## Exceptions

AMX-E4; see Section 2.10, “Intel® AMX Instruction Exception Classes,” for details.

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
