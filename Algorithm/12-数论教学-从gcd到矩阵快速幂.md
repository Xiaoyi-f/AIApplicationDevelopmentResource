# 数论：从 gcd 到矩阵快速幂（大数取模一次讲透）

> 我是小翊，19 岁，全干 AI 应用开发工程师。  
> 数论是算法里「看起来吓人、其实很有套路」的一类。核心本质一句话：**在整数域（尤其取模意义下）找规律，用数学定理把大计算量变成小计算量。**  
> 这一篇从 gcd 一路讲到矩阵快速幂、ST 表、lowbit，把大厂笔试爱考的数论技巧一次讲透。  
> Let's use AI to create a better world。

---

## 目录

- [一、数论的核心本质](#一数论的核心本质)
- [二、gcd 和 lcm](#二gcd-和-lcm)
- [三、逆元（重点）](#三逆元重点)
- [四、质数筛法（埃氏筛 + 线性筛）](#四质数筛法埃氏筛--线性筛)
- [五、快速幂](#五快速幂)
- [六、矩阵快速幂](#六矩阵快速幂)
- [七、可逆性与幂等性](#七可逆性与幂等性)
- [八、lowbit](#八lowbit)
- [九、数论速查表](#九数论速查表)

---

## 一、数论的核心本质

> **在整数域（特别是取模意义下）寻找特定规律，利用数学定理将大计算量转化为小计算量。**

**三条铁律（必背，血泪教训）**：

1. **绝对不要用浮点数处理大整数和取模**（如 10¹⁸ 级别运算），精度丢失全盘皆输。
2. **取模运算中，减法要 `(a - b + mod) % mod`**，防止出现负数。
3. **除法一定要求逆元，绝不能直接除！** 取模世界里，「除以」=「乘以逆元」。

---

## 二、gcd 和 lcm

```python
def gcd(a, b):
    """辗转相除法求最大公约数"""
    while b:
        a, b = b, a % b
    return a

def lcm(a, b):
    """求最小公倍数，先除后乘避免溢出"""
    return a // gcd(a, b) * b
```

**核心理解（易错）**：
- **辗转相除法**：`gcd(a, b) = gcd(b, a % b)`，直到 `b == 0`。
- lcm 要**先除后乘**（`a // gcd * b`），如果先乘 `a * b` 可能溢出。

---

## 三、逆元（重点）

**逆元定义**：在模 p 意义下，如果 `a × x ≡ 1 (mod p)`，那么 x 就是 a 的乘法逆元，记作 `inv(a)`。

**为什么需要逆元**：取模世界里不能直接除，要「乘以逆元」。

### 3.1 扩展欧几里得求逆元（p 不一定是质数）

```python
class Solution:
    def mod_inverse(self, a: int, p: int) -> int:
        """求 a 在模 p 下的乘法逆元（扩展欧几里得算法）"""
        def exgcd(x, y):
            # 递归：找到一组 x0,y0 满足 a*x0 + p*y0 = gcd(a,p)
            if y == 0:
                return x, 1, 0
            g, x1, y1 = exgcd(y, x % y)
            x0 = y1
            y0 = x1 - (x // y) * y1
            return g, x0, y0

        g, x, y = exgcd(a, p)
        if g != 1:
            # gcd≠1，没有逆元
            return -1
        # 把解调整为正数
        return x % p

if __name__ == "__main__":
    sol = Solution()
    print(sol.mod_inverse(2, 5))   # 输出3，2*3 mod5 = 1
    print(sol.mod_inverse(3, 7))   # 输出5，3*5 mod7 = 1
```

### 3.2 费马小定理求逆元（p 是质数）

```python
class Solution:
    def fermat_inv(self, a: int, p: int) -> int:
        """费马小定理求逆元，前提：p 是质数，a 与 p 互质"""
        # pow(a, p-2, p) 内置快速幂，计算 a^(p-2) mod p
        return pow(a, p - 2, p)

if __name__ == "__main__":
    sol = Solution()
    print(sol.fermat_inv(2, 5))   # 3
    print(sol.fermat_inv(4, 7))   # 2，4*2 mod 7 = 1
```

**两种求逆元怎么选（重点）**：

| 方法 | 条件 | 适用 |
| --- | --- | --- |
| 扩展欧几里得 | p 任意，只要 `gcd(a,p)=1` | p 不是质数 |
| 费马小定理 | p 必须是质数 | p 是质数（如 1e9+7） |

> 费马小定理：p 是质数时，`a^(p-1) ≡ 1 (mod p)`，所以 `inv(a) = a^(p-2) mod p`。

**⚠️ 核心红线**：
> 只有当 `gcd(a, p) == 1` 时，逆元才存在！如果 p 不是质数，必须用扩展欧几里得求逆元。**不要在任何除法场景用「向下取整」的常规除法，取模世界里那等于错误答案。**

---

## 四、质数筛法（埃氏筛 + 线性筛）

### 4.1 埃氏筛

```python
def sieve(n):
    """埃氏筛，找出 2~n 所有质数"""
    if n < 2:
        return []
    is_prime = [True] * (n + 1)
    is_prime[0] = is_prime[1] = False

    # 只需要循环到 sqrt(n) 就够了
    for i in range(2, int(n ** 0.5) + 1):
        if is_prime[i]:
            # 从 i*i 开始标记（i*2、i*3 已经被更小质数标记过）
            is_prime[i*i : n+1 : i] = [False] * len(is_prime[i*i : n+1 : i])

    primes = [num for num, flag in enumerate(is_prime) if flag]
    return primes

if __name__ == "__main__":
    print(sieve(20))
```

**核心理解**：
- 思想：**合数一定有一个比它小的质因子**，所以会被提前标记掉。
- 优化：从 `i*i` 开始标记（`i*2`、`i*3` 已被更小质数标记过）。
- 复杂度 O(n log log n)。

### 4.2 线性筛（欧拉筛）

```python
def linear_sieve(n: int):
    # 标记数组，初始全为 True（假设都是质数）
    is_prime = [True] * (n + 1)
    primes = []

    for y in range(2, n + 1):
        if is_prime[y]:
            primes.append(y)

        for x in primes:
            composite = y * x
            if composite > n:
                break

            is_prime[composite] = False

            # 核心优化：如果 y 能被 x 整除，说明 x 是 y 的最小质因子，必须退出
            if y % x == 0:
                break

    return primes
```

**核心理解（面试考点）**：
> 线性筛保证**每个合数只被它最小的质因子筛掉一次**，把复杂度从 O(n log log n) 压到严格的 **O(n)**。

- 关键那句 `if y % x == 0: break`：当 `x` 是 `y` 的最小质因子时，退出，保证每个合数只被最小质因子筛一次。
- Python 大量循环易超时，用 `sys.stdin` 快读 + 列表索引。

---

## 五、快速幂

```python
def fast_pow(base: int, exp: int, mod: int) -> int:
    """计算 (base ^ exp) % mod"""
    res = 1
    base = base % mod  # 防止 base 比 mod 大

    while exp > 0:
        if exp & 1:  # 如果当前二进制最低位是 1
            res = (res * base) % mod

        base = (base * base) % mod  # 底数平方
        exp >>= 1  # 指数右移一位 (相当于 exp //= 2)

    return res
```

**核心理解（易错）**：
> 快速幂利用**二进制拆分指数**，把 O(N) 的连乘降到 **O(log N)**。

- `exp & 1`：判断指数当前最低位是不是 1。
- **底数 base 每次平方后一定要取模**，指数要 `exp >>= 1`。

---

## 六、矩阵快速幂

**用途**：当递推式是线性的且 N 极大（如 10¹⁸），用矩阵快速幂代替普通 DP。

```python
MOD = 10**9 + 7

def mat_mul(A, B):
    """矩阵乘法 A * B，这里固定为 2x2 矩阵"""
    n = len(A)
    C = [[0] * n for _ in range(n)]
    for i in range(n):
        for k in range(n):
            if A[i][k] == 0: continue  # 优化：跳过 0
            for j in range(n):
                C[i][j] = (C[i][j] + A[i][k] * B[k][j]) % MOD
    return C

def mat_pow(matrix, exp):
    """矩阵快速幂，求 matrix 的 exp 次方"""
    n = len(matrix)
    # 单位矩阵（相当于数字里的 1）
    res = [[1 if i == j else 0 for j in range(n)] for i in range(n)]

    while exp > 0:
        if exp & 1:
            res = mat_mul(res, matrix)
        matrix = mat_mul(matrix, matrix)
        exp >>= 1

    return res

def fib(n: int) -> int:
    """求第 n 项斐波那契数 (项从 0 开始)"""
    if n < 2:
        return n
    # 构造转移矩阵
    base_matrix = [
        [1, 1],
        [1, 0]
    ]
    res_matrix = mat_pow(base_matrix, n - 1)
    return res_matrix[0][0] % MOD
```

**核心理解（易错）**：
- 斐波那契的转移矩阵 `[[1,1],[1,0]]`，`F(n)` = 它的 n-1 次方的 `[0][0]`。
- **矩阵乘法不满足交换律**（`A*B != B*A`），别搞错顺序。
- **单位矩阵构造绝对不能错**（对角线是 1，其余 0）。

---

## 七、可逆性与幂等性

### 7.1 可逆性（异或前缀和）

**可逆性**：一个操作做完之后，存在另一个操作可以把它抵消，恢复原数据。

```python
def xor_prefix(arr):
    """异或前缀和（可逆性典型例题）"""
    n = len(arr)
    pre = [0] * (n + 1)
    for i in range(n):
        pre[i+1] = pre[i] ^ arr[i]
    return pre

def query_xor(pre, l, r):
    """查询 [l, r] 区间异或（下标从1开始）"""
    # pre[r] ^ pre[l-1] 把前面 0~l-1 的部分抵消掉
    return pre[r] ^ pre[l - 1]

if __name__ == "__main__":
    nums = [1, 2, 3, 4]
    pre_arr = xor_prefix(nums)
    print("前缀异或数组：", pre_arr)
    print(query_xor(pre_arr, 1, 3))  # [1,2,3] 异或 1^2^3 = 0
```

> 异或性质：`x ^ y ^ y = x`，能抵消、还原——这就是可逆。**「抵消 = 可逆」的核心体现。**

### 7.2 幂等性（ST 表）

**幂等性**：同一个操作重复做很多次，结果和只做一次一样。如 `max(a,a)=a`。

```python
import math

class SparseTable:
    def __init__(self, arr):
        """ST表 区间最大值查询（幂等性典型例题）"""
        self.n = len(arr)
        self.log_table = [0] * (self.n + 1)
        for i in range(2, self.n + 1):
            self.log_table[i] = self.log_table[i // 2] + 1

        self.k_max = self.log_table[self.n] + 1
        self.st = [[0]*self.n for _ in range(self.k_max)]
        self.st[0] = arr.copy()

        # 倍增构建
        for j in range(1, self.k_max):
            for i in range(self.n - (1 << j) + 1):
                self.st[j][i] = max(
                    self.st[j-1][i],
                    self.st[j-1][i + (1 << (j-1))]
                )

    def query_max(self, l, r):
        """查询[l, r]最大值，下标从0开始"""
        length = r - l + 1
        k = self.log_table[length]
        # 两段区间会重叠，但 max 幂等，结果不变
        return max(
            self.st[k][l],
            self.st[k][r - (1 << k) + 1]
        )

if __name__ == "__main__":
    nums = [5, 3, 7, 2, 9, 1]
    st = SparseTable(nums)
    print(st.query_max(0, 4))  # [5,3,7,2,9] 最大值 9
```

**核心理解**：
- ST 表用「倍增」预处理，O(1) 查询区间最值（RMQ）。
- 查询时两段区间**会重叠**，但 `max` 幂等（`max(a,a)=a`），重叠不影响结果。
- **前提是「幂等」操作**（max、min、gcd 都幂等；sum 不幂等，因为 `sum(a,a)≠a`）。

---

## 八、lowbit

```python
def lowbit(x: int) -> int:
    """lowbit：取出x二进制最靠右的1对应的十进制数值"""
    return x & -x
```

**核心理解**：
> `lowbit(x) = x & (-x)`，利用补码特性，提取数字 x 二进制中最低位的 1 及其后面的 0。

- 是**树状数组的底层灵魂**，也用于快速枚举集合子集。
- Python 负数没有位数限制，可直接 `x & -x`（C++ 需写补码，但底层逻辑一样）。

---

## 九、数论速查表

| 技巧 | 一句话 | 复杂度 |
| --- | --- | --- |
| 辗转相除 | gcd(a,b)=gcd(b,a%b) | O(log n) |
| 逆元 | 除法转「乘逆元」 | — |
| 费马小定理 | p 质数时 inv = a^(p-2) | O(log p) |
| 埃氏筛 | 合数有更小质因子 | O(n log log n) |
| 线性筛 | 每个合数只被最小质因子筛一次 | O(n) |
| 快速幂 | 二进制拆分指数 | O(log n) |
| 矩阵快速幂 | 线性递推 + 大 N | O(log n) |
| 异或前缀 | 抵消 = 可逆 | O(n) 预处理 |
| ST 表 | 倍增 + 幂等，O(1) 查 RMQ | O(n log n) 预处理 |
| lowbit | x & -x 取最低位 1 | O(1) |

---

**一句话总结**：数论核心是「**取模世界里的数学规律**」。背熟三条红线（不用浮点、减法加 mod、除法求逆元），再掌握筛法、快速幂、逆元三大工具，大厂数论题就稳了。

**文末预告**：下一篇「背包问题」——01 背包、完全背包、多重背包，把「容量约束下选最优」这个 DP 经典模型一次讲透。

Let's use AI to create a better world。
