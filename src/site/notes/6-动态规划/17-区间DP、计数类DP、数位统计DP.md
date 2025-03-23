---
{"dg-publish":true,"permalink":"/6-动态规划/17-区间DP、计数类DP、数位统计DP/"}
---

# 区间DP、计数类DP、数位统计DP
## 前言

本讲通过三道例题来感受下区间DP、计数类DP、数位统计类DP的基本思想。
## 区间DP

### 例题 合并石子
#### 题目描述

设有 $N$ 堆石子排成一排，其编号为 $1,2,3,…,N$。

每堆石子有一定的质量，可以用一个整数来描述，现在要将这 $N$ 堆石子合并成为一堆。

每次只能合并相邻的两堆，合并的代价为这两堆石子的质量之和，合并后与这两堆石子相邻的石子将和新堆相邻，合并时由于选择的顺序不同，合并的总代价也不相同。

例如有 $4$ 堆石子分别为 `1  3  5  2`， 我们可以先合并 $1、2$ 堆，代价为 $4$，得到 `4 5 2`， 又合并 $1、2$ 堆，代价为 $9$，得到 `9 2` ，再合并得到 $11$，总代价为 $4+9+11=24$；

如果第二步是先合并 $2、3$ 堆，则代价为 $7$，得到 `4 7`，最后一次合并代价为 $11$，总代价为 $4+7+11=22$。 

问题是：找出一种合理的方法，使总的代价最小，输出最小代价。
#### 输入格式

第一行一个数 $N$ 表示石子的堆数 $N$。

第二行 $N$ 个数，表示每堆石子的质量(均不超过 $1000$)。

数据范围：$1 \le N \le 300$
#### 输出格式

输出一个整数，表示最小代价。
#### 输入样例

```
4
1 3 5 2
```
#### 输出样例

```
22
```
#### 题目分析

![Pasted image 20240321142248.png](https://s2.loli.net/2024/04/17/nkpIzKj3aUZrbyO.png)

解题思路：
关键点：最后一次合并一定是左边连续的一部分和右边连续的一部分进行合并。

最终答案：$f[1][n]$

**区间DP常用模板**

所有的区间DP问题枚举时，第一维通常是枚举区间长度，并且一般 $len = 1$ 时用来初始化，枚举从 $len = 2$ 开始；第二维枚举起点 $i$（右端点 $j$ 自动获得，$j = i + len - 1$）

```cpp
for (int len = 1; len <= n; len++) {         // 区间长度
    for (int i = 1; i + len - 1 <= n; i++) { // 枚举起点
        int j = i + len - 1;                 // 区间终点
        if (len == 1) {
            dp[i][j] = 初始值
            continue;
        }

        for (int k = i; k < j; k++) {        // 枚举分割点，构造状态转移方程
            dp[i][j] = min(dp[i][j], dp[i][k] + dp[k + 1][j] + w[i][j]);
        }
    }
}
```
#### 示例代码 时间复杂度$O(n^3)$

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;
const int N = 333;

int f[N][N], s[N];


int main()
{
    int n;
    cin >> n;
    for(int i = 1; i <= n; i++)
        cin >> s[i], s[i] += s[i - 1];
    
    for(int len = 2; len <= n; len ++) {
        for(int i = 1; i + len - 1 <= n; i++)
        {
            int j = i + len - 1;
            f[i][j] = 1e9;
            
            for(int k = i; k < j; k++)
                f[i][j] = min(f[i][j], f[i][k] + f[k + 1][j] + s[j] - s[i-1]);
        }
    }
    cout << f[1][n];
    return 0;
}
```
#### 扩展一：从后往前倒着枚举状态

除了按长度枚举，也可以倒着枚举，因为只要保证每种状态都被**提前计算**即可

从下往上倒着枚举，可以保证你算$dp[i][j]$时，$dp[i][k]$和$dp[k + 1][j]$一定是被提前计算好的，而从前往后枚举则无法保证这一点。所以我们采用倒着枚举。

```cpp
#include <iostream>

using namespace std;

const int N = 307;

int s[N];
int f[N][N];

int main() {
    int n;
    cin >> n;

    for (int i = 1; i <= n; i++) {
        cin >> s[i];
        s[i] += s[i - 1];
    }

    for (int i = n; i >= 1; i--) {  //起点
        for (int j = i; j <= n; j++) { //终点
            if (j == i) {
                f[i][j] = 0;
                continue;
            }
            f[i][j] = 1e9;
            for (int k = i; k < j; k++) {
                f[i][j] = min(f[i][j], f[i][k] + f[k + 1][j] + s[j] - s[i - 1]);
            }
        }
    }

    cout << f[1][n] << endl;

    return 0;
}
```
#### 扩展二：记忆化搜索

```cpp
#include <iostream>
#include <cstring>

using namespace std;

const int N = 307;

int a[N], s[N];
int f[N][N];

// 记忆化搜索：dp的记忆化递归实现
int dp(int i, int j) {
    if (i == j) return 0; // 判断边界
    int &v = f[i][j];

    if (v != -1) return v;

    v = 1e8;
    for (int k = i; k <= j - 1; k ++)
        v = min(v, dp(i, k) + dp(k + 1, j) + s[j] - s[i - 1]);

    return v;
}

int main() {
    int n;
    cin >> n;

    for (int i = 1; i <= n; i ++) {
        cin >> a[i];
        s[i] += s[i - 1] + a[i];
    }

    memset(f, -1, sizeof f);

    cout << dp(1, n) << endl;


    return 0;
}
```

## 计数类DP

### 例题 整数划分
#### 题目描述

一个正整数 $n$ 可以表示成若干个正整数之和，形如：$n = n_1 + n_2 + … + n_k$，其中 $n_1 \ge n_2 \ge … \ge n_k, k \ge 1$。

我们将这样的一种表示称为正整数 $n$ 的一种划分。

现在给定一个正整数 $n$，请你求出 $n$ 共有多少种不同的划分方法。
#### 输入格式

共一行，包含一个整数 $n$。

数据范围：$1 \le n \le 1000$
#### 输出格式

共一行，包含一个整数，表示总划分数量。

由于答案可能很大，输出结果请对 $10^9 + 7$ 取模。
#### 输入样例

```
5
```

#### 输出样例

```
7
```
#### 题目分析

**思路一：完全背包**

状态表示：
`f[i][j]`表示只从`1~i`中选，且总和等于`j`的方案数

状态转移方程:
`f[i][j] = f[i - 1][j] + f[i][j - i];`

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;
const int mod = 1e9 + 7;
int n, f[1010];

int main()
{
    
    cin >> n;
    f[0] = 1;
    for(int i = 1; i <= n; i++)
        for(int j = i; j <= n; j++)
            f[j] = (f[j] + f[j - i]) % mod;
        
    cout << f[n];
    
    return 0;
}
```

**思路二：其他解法**

状态表示：
`f[i][j]`表示总和为`i`，总个数为`j`的方案数

状态转移方程：
`f[i][j] = f[i - 1][j - 1] + f[i - j][j];`

![Pasted image 20240321153719.png](https://s2.loli.net/2024/04/17/XnrJPxw3CLj1HeR.png)

```cpp
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 1010, mod = 1e9 + 7;

int n;
int f[N][N];

int main()
{
    cin >> n;

    f[1][1] = 1;
    for (int i = 2; i <= n; i ++ )
        for (int j = 1; j <= i; j ++ )
            f[i][j] = (f[i - 1][j - 1] + f[i - j][j]) % mod;

    int res = 0;
    for (int i = 1; i <= n; i ++ ) res = (res + f[n][i]) % mod;

    cout << res << endl;

    return 0;
}
```
## 数位统计DP

### 例题三 计数问题
#### 题目描述

给定两个整数 $a$ 和 $b$，求 $a$ 和 $b$ 之间的所有数字中 $0 \sim 9$ 的出现次数。

例如，$a=1024，b=1032$，则 $a$ 和 $b$ 之间共有 $9$ 个数如下：

`1024 1025 1026 1027 1028 1029 1030 1031 1032`

其中 `0` 出现 $10$ 次，`1` 出现 $10$ 次，`2` 出现 $7$ 次，`3` 出现 $3$ 次等等…
#### 输入格式

输入包含多组测试数据。

每组测试数据占一行，包含两个整数 $a$ 和 $b$。

当读入一行为 `0 0` 时，表示输入终止，且该行不作处理。

数据范围：$0 < a,b < 100000000$
#### 输出格式

每组数据输出一个结果，每个结果占一行。

每个结果包含十个用空格隔开的数字，第一个数字表示 `0` 出现的次数，第二个数字表示 `1` 出现的次数，以此类推。
#### 输入样例

```
1 10
44 497
346 542
1199 1748
1496 1403
1004 503
1714 190
1317 854
1976 494
1001 1960
0 0
```
#### 输出样例

```
1 2 1 1 1 1 1 1 1 1
85 185 185 185 190 96 96 96 95 93
40 40 40 93 136 82 40 40 40 40
115 666 215 215 214 205 205 154 105 106
16 113 19 20 114 20 20 19 19 16
107 105 100 101 101 197 200 200 200 200
413 1133 503 503 503 502 502 417 402 412
196 512 186 104 87 93 97 97 142 196
398 1375 398 398 405 499 499 495 488 471
294 1256 296 296 296 296 287 286 286 247
```
#### 题目分析

暴力：枚举每一个数的每一位（共$8$位），一共要判断$10$个数，总时间复杂度是$O(10^8*8*10*T)$，$T$为$T$组数据，超时。

数位统计dp。

尤其强调**分类讨论**：

假设对于数字 $n$，我们要从中找到 $x$ 的个数。

![Pasted image 20240321182955.png](https://s2.loli.net/2024/04/17/YLj7iUqf619hSZc.png)

1.就得先求 $x$ 在每一个位置上出现的次数

2.一般性的，我们假设 $x$ 出现非第一个位置。

![Pasted image 20240321183147.png](https://s2.loli.net/2024/04/17/LUoC5geBNwPQ4Dz.png)

此时，我们假设第 $i$ 个位置，也就是 $n$ 的第 $i$ 是 $a_i$，前 $i-1$ 的值是 $k$，后 $n-i$ 位是 $t$。
那么我们分情况讨论：
- 当左边的值取 $[0,k-1]$ 时，第 $i$ 个位置取 $x$，那么后面可以取 $[0,t]$。此时贡献的数字个数为：$k*10^{t的位数}$
- 当左边的值取 $k$ 时，继续分情况讨论：
	- $a_i < x$ ，此时值已经比 $n$ 已经比这个数小了，在范围之外的，所以不贡献任何。
	- $a_i = x$，此时右边线段可以取 $[0, t]$，即贡献 $t + 1$。
	- $a_i > x$，此时 $10^{t的位数}$
#### 示例代码

```cpp
#include <iostream>  
#include <algorithm>  
#include <vector>  
  
using namespace std;  
  
const int N = 10;  
  
/*  
  
001~abc-1, 999  
  
abc  
1. num[i] < x, 0  
2. num[i] == x, 0~efg  
3. num[i] > x, 0~999  
  
*/  
  
int get(vector<int> num, int l, int r) {  
    int res = 0;  
    for (int i = l; i >= r; i -- ) res = res * 10 + num[i];  
    return res;  
}  
  
int power10(int x) {  
    int res = 1;  
    while (x -- ) res *= 10;  
    return res;  
}  
  
int count(int n, int x) {  
    if (!n) return 0;  
  
    vector<int> num;  
    while (n) {  
        num.push_back(n % 10);  
        n /= 10;  
    }  
    n = num.size();  
  
    int res = 0;  
    for (int i = n - 1 - !x; i >= 0; i -- ) { //0，不能出现在一个位置，否则可以出现在任何位置  
        if (i < n - 1) { //只要不是最后一位  
            res += get(num, n - 1, i + 1) * power10(i);//get得到思路里的k，pow得到10^{t的位数}  
            if (!x) res -= power10(i); //如果是0，前面只取1~k-1，所以要减去一个  
        }  
  
        if (num[i] == x) res += get(num, i - 1, 0) + 1;  
        else if (num[i] > x) res += power10(i);  
    }  
  
    return res;  
}  
  
int main() {  
    int a, b;  
    while (cin >> a >> b, a) {  
        if (a > b) swap(a, b);  
  
        for (int i = 0; i <= 9; i ++ )  
            cout << count(b, i) - count(a - 1, i) << ' ';  
        cout << endl;  
    }  
  
    return 0;  
}
```