# 背包问题


# 0-1 背包

> 有 N 件物品和一个容量为 V 的背包。放入第 i 件物品耗费的费用是 cost[i]， 价值是 value[i]。求解将哪些物品装入背包可使价值总和最大

```rust
//初始化
let v = 2;//背包体积
let n = 4;//物品个数,必须等于cost和value的长度
let cost = vec![1, 2, 3, 4];//物品花费
let value = vec![1, 3, 5, 7];//物品价值
let mut f = vec![0; v + 1];

// 0 <= i < n
for i in 0..n {
    zoro_one_pack(&mut f, cost[i], value[i], v);
}
//最后价值总和最大的值就是f[v]
```

```rust
fn zoro_one_pack(f: &mut Vec<usize>, cost: usize, value: usize, v: usize) {
    for j in (cost..=v).rev() {
        f[j] = std::cmp::max(f[j], f[j - cost] + value);
    }
}
```

**_注意：_**

如果题目要求 **恰好装满背包** ,则除了`f[0]`设为 0，其余都设为 $-∞$，如果**不**要求恰好装满，则`f`全初始化为 0。

### _一个常数优化_

```rust
//伪代码
for i in 0..n {
    for j in (cpm::max(cost[i],v-(cost[i]+..+cost[n-1]))..=v).rev() {
        f[j] = std::cmp::max(f[j], f[j - cost[i]] + value[i]);
    }
}
```

# 完全背包

> 有 `N` 种物品和一个容量为`V` 的背包，每种物品都有**无限件**可用。放入第 i 种物品的费用是 `cost[i]`，价值是 `value[i]`。求解：将哪些物品装入背包，可使这些物品的耗费的费用总 和不超过背包容量，且价值总和最大

思路是转化成**0-1 背包：将一种物品拆成多件只能选 0 件或 1 件的 01 背包中的物品。**

```rust
//初始化
let v = 2;//背包体积
let n = 4;//物品个数,必须等于cost和value的长度
let cost = vec![1, 2, 3, 4];//物品花费
let value = vec![1, 3, 5, 7];//物品价值
let mut f = vec![0; v + 1];

// 0 <= i < n
for i in 0..n {
    zoro_one_pack(&mut f, cost[i], value[i], v);
}
//最后价值总和最大的值就是f[v]
```

```rust
fn complete_pack(f: &mut Vec<usize>, cost: usize, value: usize, v: usize) {
    //就是将0-1背包中内层循环次序反转
		for j in cost..=v {
        f[j] = std::cmp::max(f[j], f[j - cost] + value);
    }
}
```

# 多重背包

> 有 `N` 种物品和一个容量为 `V` 的背包。第 i 种物品最多有 `M[i]` 件可用，每件耗费的空间是 `cost[i]`，价值是 `value[i]`。求解将哪些物品装入背包可使这些物品的耗费的空间总和不超 过背包容量，且价值总和最大

```rust
fn multiple_pack(f: &mut Vec<usize>, cost: usize, value: usize, v: usize, mut m: usize) {
    if cost * m >= v {
        complete_pack(f, cost, value, v);
        return;
    }
    let mut k = 1;
    while k < m {
        zoro_one_pack(f, k * cost, k * value, v);
        m -= k;
        k *= 2;
    }
    zoro_one_pack(f, m * cost, m * value, v)
}
```

### 可行性问题

> 当问题是“**每种有若干件的物品能否填满给定容量的背包**”，只须考虑填满背包的可行性，不需考虑每件物品的价值时，多重背包问题同样有 O(VN) 复杂度的算法

```rust
fn multiple_pack_ok() {
		//也可以用硬币模型来理解。v代表硬币的总价值，n代表硬币的种类，
    //m是每个硬币的数量，cost代表每种硬币的价值
    let v = 2;
    let n = 4;
    let cost = vec![5, 4, 9, 3];
    let m = vec![2, 1, 4, 6];
    let mut f = vec![-1; v + 1];
    f[0] = 0;

    for i in 0..n {
        for j in 0..=v {
            if f[j] >= 0 {
                f[j] = m[i];
            }
						//else {
            //    f[j] = -1; 应为默认初始化就是-1
            //}
        }
        if v < cost[i] {
            break;
        }
        for j in 0..=(v - cost[i]) {
            if f[j] >= 0 {
                f[j + cost[i]] = std::cmp::max(f[j + cost[i]], f[j] - 1);
            }
        }
    }
}
```

`f[i][j]`表示使用前`i`个物品，填充容量为`j`的背包，第`i`个物品最多能够剩余多少个，如果无法填充容量为`j`的背包，则值为`-1`

1. 首先，`f[i - 1][j]` 代表前 `i - 1` 件物品凑面值 `j`，如果其值大于等于 `0` 即状态合法可以凑出`j`，就说明接下来不需要第`i`种硬币就能凑出`j`，所以剩余的硬币数就是`m[i]`了。
2. 如果`f[i - 1][j]`小于`0`，说明前`i-1`种凑不出来`j`。加上第`i`个硬币可能面值太大，也可能正好，所以先取 `-1`待定。
3. `f[i][j + cost[i]] = std::cmp::max(f[i][j + cost[i]], f[i][j] - 1);` 如果能凑成`j+cost[i]`,那么就把硬币数量`-1`，如果不行，就维持`-1`的状态。

然后把二维数组改为一位数组。

# 混合背包

就是上面三种背包混合在一起

```rust
//伪代码：
for i in 0..n{
	if i 是01背包{
		zero_one_pack(..);
	}else if i 是完全背包{
		complete_pack(..);
	}else if i 是多重背包{
		multiple_pack(..);
	}
}
```

[男人八题之多重背包问题](https://zhuanlan.zhihu.com/p/56183941)

# 二维背包

> 对于每件物品，具有两种不同的费用，选择这件物品必须同时付出这两种费用。对于每种费用都有一个可付出的最大值（背包容量）。问怎样 选择物品可以得到最大的价值。

> 设第 i 件物品所需的两种费用分别为 Ci 和 Di。两种费用可付出的最大值（也即两种背包容量）分别为 V 和 U。物品的价值为 Wi

状态转移方程如下：

`F[i, v, u] = max{F[i − 1, v, u], F[i − 1, v − Ci, u − Di] +Wi}`

有时，“二维费用”的条件是以这样一种隐含的方式给出的：最多只能取 U 件物品。
这事实上相当于每件物品多了一种“件数”的费用，每个物品的件数费用均为 1，可以 付出的最大件数费用为 U

# 分组背包

> 有 N 件物品和一个容量为 V 的背包。第 i 件物品的费用是 Ci，价值是 Wi。这些物品被划分为 K 组，**每组中的物品互相冲突，最多选一件**。求解将哪些物品装入背包 可使这些物品的费用总和不超过背包容量，且价值总和最大

设 F[k, v] 表示前 k 组物品花费费用 v 能取得的最大权值，则有：

`F[k, v] = max{F[k − 1, v], F[k − 1, v − Ci] + Wi | item i ∈ group k}`

```rust
fn group_pack(f: &mut Vec<usize>, v: usize, cost: &Vec<usize>, value: &Vec<usize>) {
    let group = cost.len();
    for j in (0..=v).rev() {
        for k in 0..group {
            if j < cost[k] {
                break;
            }
            f[j] = cmp::max(f[j], f[j - cost[k]] + value[k]);
        }
    }
}

fn test_group() {
    let v = 4;
    let n = 3;
    let cost = vec![vec![1, 2], vec![3, 4], vec![5, 6]]; //三组
    let value = vec![vec![3, 2], vec![1, 5], vec![2, 4]]; //
    let mut f = vec![0; v + 1];

    for i in 0..n {
        group_pack(&mut f, v, &cost[i], &value[i]);
    }
    dbg!(&f[v]);
}
```

# 有依赖的背包问题

> 也就是说，物品 i 依赖于物品 j，表示若选物品 i，则必须选物品 j。

可以对主件 k 的“附件集合”先进行一次 01 背包，得到费用依次为 0. . .V − Ck 所有这些值时相应的最 大价值 Fk[0 . . . V − Ck]。那么，这个主件及它的附件集合相当于 V − Ck + 1 个物品的 物品组，其中费用为 v 的物品的价值为 Fk[v −Ck] +Wk，v 的取值范围是 Ck ≤ v ≤ V。 也就是说，原来指数级的策略中，有很多策略都是冗余的，通过一次 01 背包后，将主件 k 及其附件转化为 V −Ck + 1 个物品的物品组，就可以直接应用分组背包的算法解决问题了

[背包问题总结（下）](https://zhuanlan.zhihu.com/p/85783138)

```cpp
  /*
 即物品间存在依赖，比如i依赖于j，表示若选物品i，则必须选物品j
 http://acm.hdu.edu.cn/showproblem.php?pid=3449
 有很多个箱子，想买箱子中的物品必须先买下箱子，典型的依赖背包
 将不依赖其他物品的物品称为主件，依赖其他物品的物品称为附件
 我们有n个箱子，箱子里面的物品个数为cnt[i]
 那么箱子称为主件，箱子里面的物品称为附件
 那么考虑一个主件和它附件的集合，那么有2^n+1种策略，每种策略都是互斥的。所以它是分组背包问题。
 但是不能像一般的分组背包那样处理，因为组内有2^n+1种。
 但是考虑到费用相同时，只选择价值最大的。所以可以对组内的附件进行01背包，得到费用依次为v-c[i]...0的最大价值
 dp2[v-c[i]...0]

 */
 #include <stdio.h>
 #include <string.h>
 int dp[100000+10],dp2[100000+10];
 int box[55],cnt[55],price[55][11],value[55][11];
 inline int max(const int &a, const int &b)
 {
     return a < b ? b : a;
 }
 int main()
 {
     int n,v,i,j,k;
     while(scanf("%d%d",&n,&v)!=EOF)
     {
         memset(dp,0,sizeof(dp));
         for(i=1; i<=n; ++i)
         {
             scanf("%d%d",&box[i],&cnt[i]);
             memcpy(dp2,dp,sizeof(dp));
             for(j=1; j<=cnt[i]; ++j)
             {
                 scanf("%d%d",&price[i][j],&value[i][j]);
                 for(k=v-box[i]; k>=price[i][j]; --k)//附件进行01背包，每个dp2[k]对于组内的一种策略
                     dp2[k] = max(dp2[k],dp2[k-price[i][j]]+value[i][j]);
             }
             for(k=box[i];k<=v; ++k)
                 dp[k] = max(dp[k],dp2[k-box[i]]);//当容量为k时，取第i组的物品时得到的最大值和不取比较哪个大
         }
         printf("%d\n",dp[v]);
     }
     return 0;
 }
```

