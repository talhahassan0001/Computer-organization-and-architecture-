# Lab 4: Introduction to Venus Simulator

**Course:** Computer Organization & Architecture Lab (EL-2012), Fall 2026
**Student:** Talha Hassan Khan | **Roll No:** 24i-6012 | **Section:** CE-A
**Topics:** RISC-V arithmetic, R-type encoding, logical ops, `.data` / `lw`

---

## Task 1: C arithmetic to RISC-V

**Goal:** Translate `C = A + B`, `D = A - B`, `E = A * B`, `F = A * A` with A=5, B=4.

```asm
.text
.globl main
main:
    li  a0, 5          # A
    li  a1, 4          # B
    add t0, a0, a1     # C = 9
    sub t1, a0, a1     # D = 1
    mul t2, a0, a1     # E = 20
    mul t3, a0, a0     # F = 25
    li  a7, 10         # exit syscall
    ecall
```

**Result (registers):** t0 = 9, t1 = 1, t2 = 20, t3 = 25

---

## Task 2: R-type field breakdown

Layout: `funct7 | rs2 | rs1 | funct3 | rd | opcode`. Opcode for all R-type ALU ops = `0110011` = **51**.

| Instruction | funct7 | rs2 | rs1 | funct3 | rd | opcode |
|---|---|---|---|---|---|---|
| `add t0, a0, a1` | 0 | 11 | 10 | 0 | 5 | 51 |
| `sub t1, a0, a1` | 32 | 11 | 10 | 0 | 6 | 51 |
| `mul t2, a0, a1` | 1 | 11 | 10 | 0 | 7 | 51 |
| `mul t3, a0, a0` | 1 | 10 | 10 | 0 | 28 | 51 |

**Takeaway:** add, sub, mul share opcode and funct3; only **funct7** distinguishes them (`mul` comes from the M extension).

---

## Task 3: Nested expression

**Goal:** `(10 - 3) + (4 + 2) * 5` split into steps.

```asm
.text
.globl main
main:
    li  t0, 10
    li  t1, 3
    li  t2, 4
    li  t3, 2
    li  t4, 5
    sub t5, t0, t1     # 7
    add t6, t2, t3     # 6
    mul t6, t6, t4     # 30
    add s0, t5, t6     # 37
    li  a7, 10
    ecall
```

**Result:** t5 = 7, t6 = 30, **s0 = 37**

---

## Task 4: Logical operations

**Goal:** AND / OR / XOR on `0xFFF` and `0xF0F`.

```asm
.text
.globl main
main:
    li  t0, 0xfff
    li  t1, 0xf0f
    and t2, t0, t1
    or  t3, t0, t1
    xor t4, t0, t1
    li  a7, 10
    ecall
```

| Op | Hex | Decimal |
|---|---|---|
| AND | 0xF0F | 3855 |
| OR | 0xFFF | 4095 |
| XOR | 0x0F0 | 240 |

---

## Task 5: Debug given assembly (array sum)

Traced the provided assembly against its C equivalent (sum of array).
- `t0`/`t1` act as `ret`/`i`.
- `a0 + i*4` correctly indexes the word array.
- The repeated `1:` label is legal; `1f` / `1b` mean "next / previous label 1".
- Tested with `[1,2,3,4]`, result `t0 = 10`. No fix needed.

> Note: the original code listing was not included in the submitted report. Paste it here.

---

## Task 6: `.data` and `lw`

```asm
.data
table: .word 2, 4, 6, 8, 10, 12, 14, 16, 18, 20
.text
.globl main
main:
    la t0, table
    lw t1, 0(t0)       # 2
    lw t2, 4(t0)       # 4
    lw t3, 8(t0)       # 6
    lw t4, 12(t0)      # 8
    lw t5, 16(t0)      # 10
    li a7, 10
    ecall
```

**Result:** t1..t5 = 2, 4, 6, 8, 10. Offset grows by 4 because each word is 4 bytes.

---

## Issues to fix before submission

- Task 6 in the report has `lw t4, 12(t0` (missing `)`), which will not assemble.
- Task 5 code block is empty.
- Task 1 screenshot shows t0-t2 and s0-a2 only; t3 (=25) is not visible. Add a screenshot of x28.
