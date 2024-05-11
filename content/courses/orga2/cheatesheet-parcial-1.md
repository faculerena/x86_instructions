---
tags:
  - Orga2
  - cheatsheet
  - exam
title: Cheatsheet Parcial 1
aliases:
  - Cheatsheet Parcial 1
---
## Useful Links

- x86 instruction reference: [Link](https://www.felixcloutier.com/x86/)
- \[PDF\] Intel reference manual: [Link to full manual](https://cdrdv2.intel.com/v1/dl/getContent/671200)
	- [[intel_volume_1.pdf]]: Describes processors' architecture and programming environment supporting IA-32 and Intel® 64 architectures.
	- [[intel_volume_2.pdf]]: This document contains the full instruction set reference, A-Z, in one volume. It describes the format of the instructions and provides reference pages for them. A functional cross-volume table of contents, references, and index allow for easy navigation of the instruction set reference.
	- [Volume 3](https://cdrdv2.intel.com/v1/dl/getContent/671447) and [Volume 4](https://cdrdv2.intel.com/v1/dl/getContent/671098) are not needed for this exam. 
- NASM documentation: [Link](https://www.nasm.us/doc/) 
- SystemV ABI: [[sysv_abi.pdf]]
- C Convention (Spanish, from a teacher): [[convencion_c.pdf]]
- GDB cheatsheet (Spanish, from a teacher): [Link](https://macapiaggio.github.io/gdb-guide/)
- Manejo de la pila en Intel (Stack considerations in Intel)(Spanish, from a teacher): [[manejo_pila_intel.pdf]]


## Arguments and register volatility in 64 bits

### Non volatile registers
```asm 
RBX, RBP, R12, R13, R14, R15
```
### Return values
```asm
RAX ; integers, pointers
XMM0 ; floats
```
### Arguments
```asm
RDI, RSI, RDX, RCX, R8, R9 ; in order
XMM0, XMM1, ..., XMM7 ; floats
PUSH <...> ; if no more registers are available
```

### Stack
All `PUSH/SUB` needs their `POP/ADD`
When calling a function, ensure stack aligned to 16 Bytes.

## Prologue and epilogue

```asm {2-3, 7-8}
function_definition:
	PUSH rbp
	MOV rbp, rsp

	; CODE

	POP rbp
	RET
```

## Register names

![[register_names.jpeg]]

## Common functions

### strcmp

```c
int strcmp (const char* str1, const char* str2);
```
#### Parameters
- **str1** - a string
- **str2** - a string

#### Returns 
- 0 if equal
- >0 if first non-matching in `str1` is greater than that of `str2`
- <0 if first non-matching in `str1` is lower than that of `str2`

### strcpy
```c
char *strcpy(char *dest, const char *src)
```

#### Parameters
- **dest** − This is the pointer to the destination array where the content is to be copied.
    
- **src** − This is the string to be copied.

#### Returns
- This returns a pointer to the destination string dest.

### strlen

```c
size_t strlen(const char *str)
```
#### Parameters
- **str** − This is the string whose length is to be found.

#### Returns 
- Size of the string (until `\0`)
### malloc

```c
void *malloc(size_t size)
```
#### Parameters
- **size** − This is the size of the memory block, in bytes.

### free
```c
void free(void *ptr)
``` 

#### Parameters
- **ptr** − This is the pointer to a memory block previously allocated with malloc, calloc or realloc to be deallocated. If a null pointer is passed as argument, no action occurs.

### calloc


```c
void *calloc(size_t nitems, size_t size)
```
#### Parameters
- **nitems** − This is the number of elements to be allocated.
- **size** − This is the size of elements.
#### Returns
- This function returns a pointer to the allocated memory, or NULL if the request fails.

### memcpy
```c
void *memcpy(void *dest, const void * src, size_t n)
```
#### Parameters
- **dest** − This is pointer to the destination array where the content is to be copied, type-casted to a pointer of type void*.
- **src** − This is pointer to the source of data to be copied, type-casted to a pointer of type void*.
- **n** − This is the number of bytes to be copied.

## Struct packing and padding

![[memory_layout.jpeg]]


## Shuffle mask

Given an xmm\* register (starting from the right side):
```
127                                                                                                              0
  | 0xF0 | 0xE0 | 0xD0 | 0xC0 |0xB0 | 0xA0 | 0x90 | 0x80 | 0x70 | 0x60 | 0x50 | 0x40 | 0x30 | 0x20 | 0x10 | 0x00 |
```

Mask example (for `pshufb`) to destroy everything

```asm {7}
; reads in this order 
;                ------>
; so 0x00 position is destroyed by this ─┐
;                 ┌──────────────────────┘
;                 |
;                 v
mask_example: db 0x80, 0x80, 0x80, 0x80, 0x80, 0x80, 0x80, 0x80, 0x80, 0x80, 0x80, 0x80, 0x80, 0x80, 0x80, 0x80 
;                              ∧ 
;                              │
;                              └────────────┐
; and the third (0x20) is destroyed by this ┘
```

>_ASCII diagram as Image: [[mask_example.png]]_

Else, we set the value in the position given by the second half

```asm {3}
; reads in this order 
;                ------>
mask_example: db 0x00, 0x80, 0x01, 0x80, 0x02, 0x80, 0x03, 0x80, 0x04, 0x80, 0x05, 0x80, 0x06, 0x80, 0x07, 0x80 
```

With that mask, the register changes to:

```nasm {1, 12} /These are not used/ /These are zero because in the mask they were 0x80/
  | 0xF0 | 0xE0 | 0xD0 | 0xC0 | 0xB0 | 0xA0 | 0x90 | 0x80 | 0x70 | 0x60 | 0x50 | 0x40 | 0x30 | 0x20 | 0x10 | 0 00 |
  │                                                       │  │      │      │      │      │      │      │      │    
  └─────────────────These are not used────────────────────┘  │      │      │      │      │      │      │      │    
            ┌────────────────────────────────────────────────┘      │      │      │      │      │      │      │    
            │             ┌─────────────────────────────────────────┘      │      │      │      │      │      │      
            │             │             ┌──────────────────────────────────┘      │      │      │      │      │    
            │             │             │             ┌───────────────────────────┘      │      │      │      │     
            │             │             │             │             ┌────────────────────┘      │      │      │     
            │             │             │             │             │             ┌─────────────┘      │      │     
            │             │             │             │             │             │             ┌──────┘      │    
            ▼             ▼             ▼             ▼             ▼             ▼             ▼             ▼    
  | 0..0 | 0x07 | 0..0 | 0x06 | 0..0 | 0x05 | 0..0 | 0x04 | 0..0 | 0x03 | 0..0 | 0x02 | 0..0 | 0x01 | 0..0 | 0x00 |
     │             │             │            │             │             │             │              │           
     └─────────────┴─────────────┴────────────┴─────────────┴─────────────┴─────────────┴──────────────┘           
                    These are zero because in the mask they were 0x80                                             
```

Where the `0..0` are literal zeros and the other values are the ones that were in that position

>_ASCII diagram as Image: [[register_after_mask_pshufb.png]]_
#### Reminders
 - Use `0x80` to put a 0 (anything with position 7 as `bit[7]`, the most significant one, works) 
 - Use `0x0*` to put the value in `0x0*` into the position in the mask 

## Blend mask

Similar to shuffle, if the position in the mask has a `1`, the value in `dest` gets replaced by the one in that position in `src`, else it's unchanged.

```c
// Intel manual pseudocode
IF (imm8[0] = 1) THEN DEST[15:0] := SRC[15:0]
ELSE DEST[15:0] := DEST[15:0]
IF (imm8[1] = 1) THEN DEST[31:16] := SRC[31:16]
ELSE DEST[31:16] := DEST[31:16]
IF (imm8[2] = 1) THEN DEST[47:32] := SRC[47:32]
ELSE DEST[47:32] := DEST[47:32]
IF (imm8[3] = 1) THEN DEST[63:48] := SRC[63:48]
ELSE DEST[63:48] := DEST[63:48]
IF (imm8[4] = 1) THEN DEST[79:64] := SRC[79:64]
ELSE DEST[79:64] := DEST[79:64]
IF (imm8[5] = 1) THEN DEST[95:80] := SRC[95:80]
ELSE DEST[95:80] := DEST[95:80]
IF (imm8[6] = 1) THEN DEST[111:96] := SRC[111:96]
ELSE DEST[111:96] := DEST[111:96]
IF (imm8[7] = 1) THEN DEST[127:112] := SRC[127:112]
ELSE DEST[127:112] := DEST[127:112]
``` 

The mask in blend is `imm8`, so for example `01010101` (read from right to left) would leave the odd values equal to `dest`, and even values replaced by `src`

## NASM Size specifiers
- BYTE | WORD | DWORD | QWORD | TWORD | OWORD | YWORD | ZWORD ([Source](https://www.nasm.us/xdoc/2.16.03/html/nasmdoc3.html#section-3.1)) 

## Solved problems
- "Mirá que coincidencia" (greyscale pixels, blend, float to int, int to float, pack dword to word, word to byte): https://godbolt.org/z/Exaznxb78

## Compare pixels
(Tags: pixels, `pcmpgtb`)

![[compare_pixels.jpeg]]