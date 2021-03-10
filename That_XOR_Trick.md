# That XOR Trick

## XOR

| `x`  | `y`  | `x ^ y` |
| :--: | :--: | :-----: |
|  0   |  0   |    0    |
|  0   |  1   |    1    |
|  1   |  0   |    1    |
|  1   |  1   |    0    |

## 公式

- `x ^ 0 = x`
- `x ^ x = 0`

## Commutativity (交换律)

**XOR** 操作满足交换律

```
  a ^ b ^ c ^ a ^ b    # commutativity
= a ^ a ^ b ^ b ^ c    # Using x ^ x = 0
= 0 ^ 0 ^ c            # Using x ^ 0 = x
= c
```

## Application 1: In-Place Swapping

交换两个变量的值. 假设: x 和 y 的值分别是 a 和 b

```
x = x ^ y    # (a ^ b, b)
y = x ^ y    # (a ^ b, a ^ b ^ b) ===> (a ^ b, a)
x = x ^ y    # (a ^ b ^ a, a) ===> (b, a)
```

## Application 2: Finding the Missing Number

> You are given an array `A` of n -1 integers which are in the range between 1 and n. All number appear exactly once, except on number, which is missing. Find this missing number.

```
missingNumber = 1 ^ 2 ^ 3 ^ ... ^ n ^ A[0] ^ A[1] ^ ... ^ A[n-2]
```

Note: that `A[n-2]` is the last index of a list of `n-1` elements.

> **Source**: [XOR Trick](https://florian.github.io/xor-trick/)