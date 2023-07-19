---
title: Exponential Golomb coding
author: Jack Lin
date: 2021-01-08 20:55:00 +0800
categories: [blog, C]
tags: [c, data structure]
pin: false
---

## Exponential Golomb coding

Excerpted from Wikipedia, [Exponential-Golomb coding](https://en.wikipedia.org/wiki/Exponential-Golomb_coding) (or Exp-Golomb code) is a kind of [Universal code](https://en.wikipedia.org/wiki/Universal_code_(data_compression)), that is, Exp-Golomb code can be mapped to all positive integers.

Assuming the input is `x`, the encoding steps are:

1. Write `x + 1` in binary
2. Count the binary digit of the number `x + 1` have in total, and add as many `0` as its binary digit minus 1 in front of `x + 1`.

**Example 1:**

```
x = 5
1. Write down 5 + 1 = 6 as binary "110".
2. "110" has 3 digits, we write 2 "0"s preceding "110".
   The Exp-Golomb code of 5 is "00110".
```

**Example 2:**

```
x = 24
1. Write down 24 + 1 = 25 as binary "11001".
2. "11001" has 5 digits, we write 4 "0"s preceding "11001".
   The Exp-Golomb code of 24 is "000011001".
```

Calculate the Exp-Golomb code of 0 ~ 8 according to the rules mentioned above:

```
 x   step 1   step 2
 0 ⇒  1     ⇒ 1
 1 ⇒  10    ⇒ 010
 2 ⇒  11    ⇒ 011
 3 ⇒  100   ⇒ 00100
 4 ⇒  101   ⇒ 00101
 5 ⇒  110   ⇒ 00110
 6 ⇒  111   ⇒ 00111
 7 ⇒  1000  ⇒ 0001000
 8 ⇒  1001  ⇒ 0001001
```

If you want to encode **signed** integers, just map each signed number to an unsigned number:

```
 x   map    step 1  step 2
 0 ⇒  0  ⇒  1     ⇒ 1
 1 ⇒  1  ⇒  10    ⇒ 010
−1 ⇒  2  ⇒  11    ⇒ 011
 2 ⇒  3  ⇒  100   ⇒ 00100
−2 ⇒  4  ⇒  101   ⇒ 00101
 3 ⇒  5  ⇒  110   ⇒ 00110
−3 ⇒  6  ⇒  111   ⇒ 00111
 4 ⇒  7  ⇒  1000  ⇒ 0001000
−4 ⇒  8  ⇒  1001  ⇒ 0001001
```

We can group every $2^i$ Exp-Golomb codes in order. For each code, there are $i$ `0`s in front, which means that the code belongs to $S_i$. The red `1` in the center of each code is a dividing point, and the code of the suffix is expressed in binary as the $j$th element of $S_i$, recorded as $G_{i,j}$ :

$ \\ S_0 \quad \quad \quad S_1 \quad \quad \quad \quad \quad \quad \quad \quad \\ \\ S_2 \\\\ \\{\color{red}1\\},\\ \\{0\color{red}10,\\ 0\color{red}11\\},\\ \\{00\color{red}100,\\ 00\color{red}101,\\ 00\color{red}110,\\ 00\color{red}111\\},\\  ... \\\\ G_{0,0} \\ \\ \\ G_{1,0} \\ \\ \\ G_{1,1} \quad \quad G_{2,0} \quad \\ \\ G_{2,1} \quad \\ \\ G_{2,2} \quad \\ G_{2,3}$