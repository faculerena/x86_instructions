# XSUSLDTRK**Suspend Tracking Load Addresses**

| Opcode/Instruction    | Op/En | 64/32 bit Mode Support | CPUID Feature Flag | Description                                                               |
| --------------------- | ----- | ---------------------- | ------------------ | ------------------------------------------------------------------------- |
| F2 0F 01 E8 XSUSLDTRK | ZO    | V/V                    | TSXLDTRK           | Specifies the start of an Intel TSX suspend read address tracking region. |

## Instruction Operand Encoding

| Op/En | Tuple | Operand 1 | Operand 2 | Operand 3 | Operand 4 |
| ----- | ----- | --------- | --------- | --------- | --------- |
| ZO    | N/A   | N/A       | N/A       | N/A       | N/A       |

### Description

The instruction marks the start of an Intel TSX (RTM) suspend load address tracking region. If the instruction is used inside a transactional region, subsequent loads are not added to the read set of the transaction. If the instruction is used inside a suspend load address tracking region it will cause transaction abort.

If the instruction is used outside of a transactional region it behaves like a NOP.

Chapter 16, “Programming with Intel® Transactional Synchronization Extensions‚” in the Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 1 provides additional information on Intel® TSX Suspend Load Address Tracking.

### Operation

#### XSUSLDTRK

```
IF RTM_ACTIVE = 1:
    IF SUSLDTRK_ACTIVE = 0:
        SUSLDTRK_ACTIVE := 1
    ELSE:
        RTM_ABORT
ELSE:
    NOP

```

### Flags Affected

None.

### Intel C/C++ Compiler Intrinsic Equivalent

```
XSUSLDTRK void _xsusldtrk(void);

```

### SIMD Floating-Point Exceptions

None.

### Other Exceptions

| #​​​UD                      | If CPUID.(EAX=7, ECX=0):EDX.TSXLDTRK[bit 16] = 0. |
| --------------------------- | ------------------------------------------------- |
| If the LOCK prefix is used. |

This UNOFFICIAL, mechanically-separated, non-verified reference is provided for convenience, but it may be
incomplete or broken in various obvious or non-obvious
ways. Refer to [Intel® 64 and IA-32 Architectures Software Developer’s Manual](https://software.intel.com/en-us/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4) for anything serious.
