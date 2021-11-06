# Computer Principles

## Two's Complement

> [tow's complement](https://en.wikipedia.org/wiki/Two%27s_complement)

二进制补码 (tow's complement) 是对二进制数的数学运算, 是基数补码 (radix complement) 的一个例子.

它最常用于带符号整数, 比如: Int3 类型, 其中的 `3` 表示用三位二进制数字来表示整数. 在无符号整数中, 它可以表示 `0~7` 这 8 个数字, 如果用带符号整数则其表示 `-4~3` 这 8 个数字. 那么现在的问题是 `-4` 和 `3` 是怎么来的呢?

这里我们首先要理解补码的计算方法, 即"取反加一". 比如: `010` 取反为 `101` 然后加一为 `110`. 就是说我们对 `010` 进行补码运算后的结果为 `110`.

> 如果我们在仔细看一下 `010` 转换成十进制数字是 `2`, 其补码 `110` 的十进制是 `6`, 相加刚好是 `010 + 110 = 1000` 就是十进制的 `8`.

在编程的大部分语言中, 对于带符号整数均采用补码的方式来表示符号. 我们还用三位二进制来做说明.

|三位二进制 | 十进制值 | 补码 | 补码表示的十进制值 |
| :-------:| :-----: | :--: | :---------------: |
000 | 0 | - | - |
001 | 1 | 111 | -1 |
010 | 2 | 110 | -2 |
011 | 3 | 101 | -3 |
100 | -4 | 100 | -4 |

将上表的后两位移动到前两列的下面(注意 0 是不用移动的, 因为其补码超过了三位, 也就是说 0 没有补码)

|三位二进制 | 十进制值 |
| :-------:| :-----: |
000 | 0 |
001 | 1 |
010 | 2 |
011 | 3 |
100 | -4 |
101 | -3 |
110 | -2 |
111 | -1 |

上面需要特别关注的是 `000` 这个二进制数是没有补码的, 因此可以这么理解没有"负数零"这个数, 这对计算机计算时很重要, 这也是为什么使用补码的方式来表示符号.

其次是 `100` 这个二进制数, 它的补码还是 `100`, 我们约定其为 $-4$, 而不是 $+4$, 这样在程序处理时会有便利, 这就保持了二进制由 $1$ 开头的均为负数这一约定.

容易出现的误解: `101` 应该是 `5` 啊, 就算第一位表示符号位那么后两位 `01` 是 `1`, `101` 应该表示为 `-1`, 为什么会是 `-3` 呢. 这就是因为背后是补码原理. 当我们遇到 `101` 这个二进制数时, 首先根据首位是 `1`, 从而可以判断其表示的是负数, 然后将其取补码即 `011` 是 `3`, 所以 `101` 表示的是 `-3`.

总结, 在带符号整数中, 使用二进制补码来标示其符号的反转, 二进制中首位是 `0` 表示正数, `1` 表示负数.