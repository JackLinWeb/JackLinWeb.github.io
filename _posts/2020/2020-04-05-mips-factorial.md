---
title: Implement factorial using MIPS
image: /assets/img/default-banner.jpg
author: jack
date: 2020-04-05 20:55:00 +0800
categories: [blog]
tags: [mips, assembly]
math: true
pin: false
---

This was a computer organization class assignment. I implemented a factorial using the following algorithm:

$
F(n) = \begin{cases}  1, & n = 0 \\\\ F(n - 1) * n, & n \in \mathbb{N} \end{cases}
$

In MIPS, I use `$a0` as the parameter *n* of *F(n)*. For example, letting `$a0` equal to `5` and jumping to the `fact` label means *F(5)*, and at the end of the recursion, the result will be stored in `$v0`.

```py
main:
    addi $a0, $zero, 5        # let the parameter n be 5
    jal fact                  # jump to fact label, i.e. calling F(5)
    j exit

fact:
    addi $sp, $sp, -8         # allocate 8 bytes to this stack
    sw $ra, 0($sp)            # save the address of the instruction that calls fact label (instruction address)
    sw $a0, 4($sp)            # save the value of n

    slti $t0, $a0, 1          # $t0 is used for conditions. If n < 1 then $t0 = 1, else $t0 = 0
    beq $t0, $zero, L1        # if $t0 == 0 then jump to branch L1
    addi $v0, $zero, 1        # let $v0 be 1
    addi $sp, $sp, 8          # let $sp point to upper stack
    jr $ra                    # jump to the next instruction of the instruction calling fact

L1:
    addi $a0, $a0, -1         # n = n - 1
    jal fact                  # jump to fact label again, like as calling F(n - 1)
    lw $a0, 4($sp)            # recover the value of n
    mul $v0, $a0, $v0         # $v0 *= $a0, like as F(n) = n * F(n - 1)
    lw $ra, 0($sp)            # recover instruction address
    addi, $sp, $sp, 8         # let $sp point to the upper stack
    jr $ra                    # jump to the next instruction of the instruction calling L1

exit:
```

The execution result of the MARS simulator looks like this:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1willvg4t43u7hisghz3.jpg)

You can see that the value of `$v0` is `0x78`, 120 in decimal, which is the result of *F(5)*.