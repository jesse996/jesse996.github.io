# 算法


## 图

### 无向图

-   图的表示方法：邻接表
-   dfs 和 bfs 的区别：dfs 是用栈，bfs 用队列

```java
//连通图
public class CC {
    private boolean[] marked;
    private int[] id;
    private int count;

    public CC(Graph G) {
        marked = new boolean[G.V()];
        id = new int[G.V()];
        for (int s = 0; s < G.V(); s++) {
            dfs(G, s);
            count++;
        }
    }

    private void dfs(Graph G, int v) {
        marked[v] = true;
        id[v] = count;
        for (int w : G.adj(v))
            if (!marked[w])
                dfs(G, w);

    }
}

```

### 有向图

-   有向无环图(DAG): 不含有向环的有向图
-   当且仅当一副有向图是无环图时它才能进行拓扑排序

-   有向图中基于 dfs 的顶点排序：前序、后续、逆后续
    前序和后续用队列，逆后续用栈
-   一副有向无环图的拓扑排序就是所有顶点的逆后续排列（要先判断有没有环）
-   强连通 ：两个顶点互联可达，则这两个顶点是强连通。若一个图任意两顶点都是强连通，则这幅有向图也是强连通的。
-   计算强连通分量的 Kosaraju 算法：先使用 dfs 查找 G 的反向图，得到所有顶点的逆后续，再用 dfs 处理，即可得到强连通分量

```java
//强连通分量
public class KosarajuSCC {
    private boolean[] marked;
    private int[] id;
    private int count;

    public KosarajuSCC(Digraph G) {
        marked = new boolean[G.V()];
        id = new int[G.V()];
        DepthFirstOrder order=new DepthFirstOrder(G.reverse());
        for (int s:order.reversePost()) {
            dfs(G, s);
            count++;
        }
    }

    private void dfs(Digraph G, int v) {
        marked[v] = true;
        id[v] = count;
        for (int w : G.adj(v))
            if (!marked[w])
                dfs(G, w);
    }
}
```

## 排序

### 插入排序

```rust
fn insert<T: Ord + Copy>(a: &mut [T]) {
    for i in 1..a.len() {
        let tmp = a[i];
        let mut j = i;
        while j > 0 && tmp < a[j - 1] {
            a[j] = a[j - 1];
            j -= 1;
        }
        a[j] = tmp;
    }
}
```

### 希尔排序

```rust
fn shell<T: Ord + Copy>(a: &mut [T]) {
    let n = a.len();
    let mut h = 1;
    while h < n / 3 {
        h = h * 3 + 1;
    }
    while h >= 1 {
        for i in h..n {
            let tmp = a[i];
            let mut j = i;

            while tmp < a[j - h] {
                a[j] = a[j - h];
                j -= h;
                if let None = j.checked_sub(h) {
                    break;
                }
            }
            a[j] = tmp;
        }
        h /= 3;
    }
}
```

### 归并排序

```rust
//原地归并
fn merge_help<T: Ord + Copy>(a: &mut [T], lo: usize, mid: usize, hi: usize) {
    if a[mid] <= a[mid + 1] {
        return;
    }
    let mut tmp = a.to_owned();
    //tmp.copy_from_slice(a);
    let mut i = lo;
    let mut j = mid + 1;
    for k in lo..=hi {
        if i > mid {
            a[k] = tmp[j];
            j += 1;
        } else if j > hi {
            a[k] = tmp[i];
            i += 1;
        } else if tmp[j] < tmp[i] {
            a[k] = tmp[j];
            j += 1;
        } else {
            a[k] = tmp[i];
            i += 1;
        }
    }
}

//自顶向下的归并排序
fn merge_sort<T: Ord + Copy>(a: &mut [T], lo: usize, hi: usize) {
    if hi <= lo {
        return;
    }
    //数组较小时用插入排序更快
    if hi - lo < 15 {
        insert(&mut a[lo..=hi])
    }
    let mid = (lo + hi) / 2;
    merge_sort(a, lo, mid);
    merge_sort(a, mid + 1, hi);
    merge_help(a, lo, mid, hi);
}

fn merge<T: Ord + Copy>(a: &mut [T]) {
    merge_sort(a, 0, a.len() - 1);
}

//自底向上的归并
fn merge_sort_bu<T: Ord + Copy>(a: &mut [T], lo: usize, hi: usize) {
    let n = a.len();
    let mut sz = 1;
    while sz < n {
        let mut lo = 0;
        while lo < n - sz {
            merge_help(a, lo, lo + sz - 1, std::cmp::min(lo + sz + sz - 1, n - 1));
            lo += 2 * sz;
        }
        sz *= 2;
    }
}
```

### 堆排序

```rust
#[derive(Debug)]
struct MaxPQ<T: Ord + Copy> {
    pq: Vec<T>,
}

impl<T: Ord + Copy + std::fmt::Debug> MaxPQ<T> {
    fn new(zero: T) -> Self {
        let pq = vec![zero];
        Self { pq }
    }
    fn is_empty(&self) -> bool {
        self.pq.len() <= 1
    }
    fn insert(&mut self, v: T) {
        self.pq.push(v);
        let n = self.pq.len() - 1;
        self.swim(n);
    }
    fn swim(&mut self, mut k: usize) {
        while k > 1 && self.pq[k] > self.pq[k / 2] {
            self.pq.swap(k / 2, k);
            k /= 2;
        }
    }
    fn sink(&mut self, mut k: usize, N: usize) {
        let tmp = self.pq[k];
        while k * 2 <= N {
            let mut j = k * 2;
            if j < N && self.pq[j] < self.pq[j + 1] {
                j += 1;
            }
            if self.pq[k] >= self.pq[j] {
                break;
            }
            self.pq[k] = self.pq[j];
            k = j;
        }
        self.pq[k] = tmp;
    }
    fn sort(a: &mut [T]) -> Vec<T> {
        let mut n = a.len();
        let mut pq = MaxPQ::new(a[0]);
        pq.pq.append(&mut a.to_vec());

        for k in (1..=(n / 2)).rev() {
            MaxPQ::sink(&mut pq, k, n);
        }

        while n > 1 {
            pq.pq.swap(1, n);
            n -= 1;
            MaxPQ::sink(&mut pq, 1, n);
        }
        pq.pq.remove(0);
        pq.pq
    }
}
```

Api：

```java
public class UF{
  UF(int N);//初始化N个触点
  void union(int p,int q) //在p和q之间添加一条连接
  int find(int p) // p所在的分量的标识符
  boolean connected(intp ,int q)//如果q和p在同一各分量中则返回true
  int count()//联通分量的数量
}
```

### 快速排序

```rust
fn quick_sort(a:&[i32],lo:i32,hi:i32){
}
```

```rust
void getNext(char * p, int * next)
{
//next.length=p.length
	next[0] = -1;
	int i = 0, j = -1;

	while (i < strlen(p)-1)
	{
		if (j == -1 || p[i] == p[j])
		{
			++i;
			++j;
			//next[i] = j;
			//下面是优化
			if(p[i] != p[j])
				next[i] = j;
			else
				next[i] = next[j];
		}
		else
			j = next[j];
	}
}

int KMP(char * t, char * p)
{
	int i = 0;
	int j = 0;

	while (i < strlen(t) && j < (int)strlen(p))
	{
		if (j == -1 || t[i] == p[j])
		{
							i++;
           		j++;
		}
	 	else
           		j = next[j];
   }

    if (j == strlen(p))
       return i - j;
    else
       return -1;
}
```

## 加权 quick-union 算法：

**将小数的根节点连接到大树的根节点**

```java
public class WeightedQuickUnionUF{
    private int[] id;
    private int[] sz;
    private int count;

    public WeightedQuickUnionUF(int N) {
        count = N;
        id = new int[N];
        for (int i = 0; i < N; i++) id[i] = i;
        sz = new int[N];
        for (int i = 0; i < N; i++) sz[i] = 1;
    }

    public int getCount() {
        return count;
    }

    public boolean connected(int p, int q) {
        return find(p) == find(q);
    }

    public int find(int p) {
        while (p != id[p]) p = id[p];
        return p;
    }

    public void union(int p, int q) {
        int i = find(p);
        int j = find(q);
        if (i == j) return;
        if (sz[i] < sz[j]) {
            id[i] = j;
            sz[j] += sz[i];
        } else {
            id[j] = i;
            sz[i] += sz[j];
        }
        count--;
    }
}
```

## 最优解法：**路径压缩的加权 quick-union 算法**

要实现路径压缩，只需要为`find()`添加一个循环，将在路径上遇到的所有节点都直接链接到根节点。

```java
    public int find(int p) {
        int root = p;
        while (root != id[root]) root = id[root];
        while (p!=root) {
            int next = id[p];
            id[p] = root;
            p = next;
        }
        return root;
    }
```

## BM 算法原理

[字符串匹配 ---- BM 算法原理](https://zhuanlan.zhihu.com/p/63596339)

