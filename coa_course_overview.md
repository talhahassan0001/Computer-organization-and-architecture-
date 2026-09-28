# Computer Organization & Architecture Lab (EL-2012), Course Overview

**NUCES Islamabad | Fall 2026**
**Instructors:** Dr. Nasir Ali Shah, Engr. Fasih Ahmad
**Student:** Talha Hassan Khan (24i-6012, CE-A)
**Tooling:** Venus RISC-V simulator (RV32IM)

## Lab index

| Lab | Topic | Notes file |
|---|---|---|
| 4 | Intro to Venus, arithmetic, R-type encoding, logic ops, `.data`/`lw` | [lab04_intro_to_venus.md](lab04_intro_to_venus.md) |
| 5 | Branches, jumps, pseudo-instructions, loops, shifts, arrays | [lab05_branches_and_loops.md](lab05_branches_and_loops.md) |

---

## Registers (ABI names)

| Reg | ABI | Role |
|---|---|---|
| x0 | zero | Constant 0 |
| x1 | ra | Return address |
| x2 | sp | Stack pointer |
| x3 / x4 | gp / tp | Global / thread pointer |
| x5-x7, x28-x31 | t0-t6 | Temporaries |
| x8-x9, x18-x27 | s0-s11 | Saved |
| x10-x17 | a0-a7 | Args / return / syscall number (a7) |

## Instruction formats

| Format | Fields (MSB to LSB) | Examples |
|---|---|---|
| R | funct7, rs2, rs1, funct3, rd, opcode | add, sub, mul, and, or, xor |
| I | imm[11:0], rs1, funct3, rd, opcode | addi, slli, lw, jalr |
| S | imm[11:5], rs2, rs1, funct3, imm[4:0], opcode | sw |
| B | imm bits, rs2, rs1, funct3, imm bits, opcode | beq, bne, blt, bge |
| J | imm[20\|10:1\|11\|19:12], rd, opcode | jal |

Opcode for R-type ALU = `0110011` (51). `sub` = funct7 32, `mul` = funct7 1.

## Pseudo-instructions seen

| Pseudo | Real |
|---|---|
| `li rd, imm` | `addi rd, x0, imm` (or `lui` + `addi`) |
| `la rd, label` | `auipc` + `addi` |
| `j L` | `jal x0, L` |
| `bgt a, b, L` | `blt b, a, L` |
| `ble a, b, L` | `bge b, a, L` |

## Core patterns

**If/else**
```asm
bne  a, b, ELSE
# if-body
j    DONE
ELSE:
# else-body
DONE:
```

**For / while loop**
```asm
LOOP:
bge  i, limit, END
# body
addi i, i, 1
j    LOOP
END:
```

**Array element address** (word array): `slli t, i, 2` then `add t, base, t`, then `lw` / `sw 0(t)`.

**Exit in Venus:** `li a7, 10` then `ecall`.

## Key concepts

- Branches are conditional (PC-relative); `j` is unconditional (`jal x0`).
- Left shift by n = multiply by 2^n.
- Word = 4 bytes, so array offsets step by 4.
- `.data` starts at `0x10000000` in Venus; `sp` starts at `0x7FFFFFDC`.
- Numeric labels (`1:`, `1f`, `1b`) may repeat; named labels may not.
- Assembler is case-sensitive for labels (`end` and `End` differ).

## Common mistakes checklist

- Missing `#` on comment lines.
- Duplicate label names in one file.
- Missing `)` in `lw rd, off(rs)`.
- Reusing registers across "parts" without resetting.
- Screenshot registers not matching the registers the code actually uses.
- No exit `ecall` at end of program.

## Workflow per lab

1. Write code, assemble, step through.
2. Screenshot the relevant registers/memory (not just the first page of the register list).
3. Fill Description / Code / Output boxes, each in its own box.
4. Cross-check expected values by hand.
