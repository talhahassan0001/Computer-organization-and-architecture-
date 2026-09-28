# Lab 5: Venus Simulator, Branches, Jumps and Loops

**Course:** Computer Organization & Architecture Lab (EL-2012), Fall 2026
**Student:** Talha Hassan Khan | **Roll No:** 24i-6012 | **Section:** CE-A
**Topics:** branch/jump instructions, pseudo-instructions, if/else, for/while, nested loops, shifts, array stores

---

## Task 1: Actual vs pseudo instructions

| Instruction | Type | Translates to |
|---|---|---|
| `beq`, `bne`, `blt`, `bge` | Actual (B-type) | n/a |
| `ble rs1, rs2, L` | Pseudo | `bge rs2, rs1, L` |
| `bgt rs1, rs2, L` | Pseudo | `blt rs2, rs1, L` |
| `jal`, `jalr` | Actual | n/a |
| `j L` | Pseudo | `jal x0, L` |

Verify in Venus: assemble, then read the **Basic Code** column of the Editor/Execute tab.

---

## Task 2: `beq` (conditional) vs `j` (unconditional)

```asm
main:
    addi s0, x0, 5
    addi s1, x0, 5
    beq  s0, s1, skip     # taken (5 == 5)
    addi s2, x0, 100      # skipped
skip:
    addi s3, x0, 200      # runs
    j    end              # always taken
    addi s4, x0, 300      # never reached
end:
    addi s5, x0, 400
```

**Result:** s0=5, s1=5, s2=0, s3=200, s4=0, s5=400.
Machine code check: `beq x8 x9 8` = `0x00940463`, `j end` = `jal x0 8` = `0x0080006F`.

---

## Task 3: If/else with not-equal path

```asm
    addi s0, s0, 4
    addi s1, s1, 6
    beq  s0, s1, Fin      # 4 != 6, not taken
True:
    add  s2, s1, s0       # s2 = 10
    j    end
Fin:
    sub  s2, s1, s0
end:
```

**Result:** s0=4, s1=6, s2=10.

**Part 2** adds 4 to both registers and expects equality. It must be run as a separate program, otherwise s0=8, s1=10 carry over and the branch is not taken. It also needs unique label names.

---

## Task 4: `if (a==b) c=d-e; else c=d+e;`

a=5, b=10, d=4, e=3.

```asm
main:
    addi s0, x0, 5
    addi s1, x0, 10
    addi s2, x0, 4
    addi s3, x0, 3
    bne  s0, s1, ELSE
    sub  s4, s2, s3
    j    DONE
ELSE:
    add  s4, s2, s3       # taken path
DONE:
```

**Result:** s4 = 7.

---

## Task 5: Shift left twice

```asm
main:
    addi s0, x0, 2
    slli s0, s0, 1        # 4
    slli s0, s0, 1        # 8
```

**Result:** s0 = 8. Left shift by n = multiply by 2^n, so two shifts = x4.

---

## Task 6: `for (i=1; i<=5; i++) counter++`

```asm
main:
    addi s0, x0, 0        # counter
    addi s1, x0, 1        # i
LOOP:
    addi t0, x0, 5
    bgt  s1, t0, END      # exit when i > 5
    addi s0, s0, 1
    addi s1, s1, 1
    j    LOOP
END:
```

**Result:** s0 = 5, t0 = 5. (The original submission had a redundant `bgt s1, x0, CONTINUE` placeholder; it is removed above since it always falls through.)

---

## Task 7: `while (i<5) num[i] = i+5;`

```asm
.data
num: .word 0,0,0,0,0,0,0,0,0,0
.text
main:
    la   s0, num
    addi s1, x0, 0
WHILE:
    addi t0, x0, 5
    bge  s1, t0, END
    addi t1, s1, 5        # value
    slli t2, s1, 2        # byte offset = i*4
    add  t3, s0, t2       # &num[i]
    sw   t1, 0(t3)
    addi s1, s1, 1
    j    WHILE
END:
```

**Result:** num[0..4] = 5, 6, 7, 8, 9. Check in the Venus **Memory** tab at `0x10000000`.

---

## Task 8: Nested loops

```asm
main:
    addi s0, x0, 0        # counter
    addi s1, x0, 0        # x
OUTER:
    addi t0, x0, 3
    bge  s1, t0, END_OUTER
    addi s2, x0, 0        # reset y each outer pass
INNER:
    addi t1, x0, 2
    bge  s2, t1, END_INNER
    add  s0, s1, s2       # counter = x + y (overwrites, not accumulates)
    addi s2, s2, 1
    j    INNER
END_INNER:
    addi s1, s1, 1
    j    OUTER
END_OUTER:
```

**Result:** last write is x=2, y=1, so s0 = 3.

---

## Issues to fix before submission

- Task 3 Part 2: `PART#2` is not a comment (needs `#`), labels `True`/`Fin`/`end` are duplicated, and registers are not reset.
- Task 2 and 6: the "Code" box says NONE / the code sits under "Output". Move it.
- Task 4 and 5: screenshots show `t0`, `t1`, `0x1000...` values that do not match the code (which uses `s` registers). Re-capture s4 and s0.
- Task 7: no memory-tab screenshot proving the array contents.
- Tasks 7 and 8: no exit `ecall`; add `li a7, 10` / `ecall` to end cleanly.
