# 网络流量算法


## 定义

流量网络：一个流量网络是一张边的权重（这里称为容量）为正的加权有向图。一个 st-流量网络有两个已知的顶点，即起点 s 和终点 t

流量配置：由一组和每条边相关的值组成的集合，这个值被称为边的流量。如果所有边的流量均小于边的容量且满足每个顶点的局部平衡（即净流量均为 0，s 和 t 除外），那么就称这种流量配置方案是可行的。

## 检查流量网络中的一种流量配置是否可行

```java
private boolean localEq(FlowNetwork G,int v){
    double EPSILON = 1E-11;
    double netflow = 0.0;
    for (FlowEdge e: G.adj(v)){
        if (v == e.from())
            netflow -= e.flow();
        else
            netflow += e.flow();
    }
    return Math.abs(netflow) < EPSILON;
}

private boolean isFeasible(FlowNetwork G){
    for (int v = 0; v < G.V(); v++){
        for(e.flow() < 0 || e.flow() > e.capacity())
            return false;
    }

    for (int v = 0; v < G.V(); v++)
        if (v != s && v != t && !localEq(v))
            return false;
    return true;
}
```

## Ford-Fulkerson 最大流量算法

网络中的初始流量为零，沿着任意从起点到终点（且不含有饱和的长相边或者是空的逆向边）的增广路径增大流量，知道网络中不存在这样的路径为止。

## 最大流-最小切分定义

### 定义

st-切分是一个将顶点 s 和顶点 t 分配于不同集合中的切分。

一个 st-切分的容量为该切分的 st-边的容量之和

st-切分的跨切分流量为该切分的 st-边的流量之和于 ts-边的流量之和的差

最小 st-切分：给定一个 st-网络，找到容量最小的 st-切分

### 定理

-   对于任意 st-流量网络，每种 st-切分中的跨切分流量都和总流量的值相等
-   s 的流出量等于 t 的流入量（即 st-流量网络的值）
-   st-流量网络的值不可能超过任意 st-切分的容量

令 f 为一个 st-流量网络，一下三种条件是等价的：

1. 存在某个 st-切分，其容量和 f 的流量相等
2. f 达到了最大流量
3. f 中已经不存在任何蹭广路径

当所有容量均为整数时，存在一个整数值的最大流量，而 Ford-Fulkerson 算法能够找出这个最大值

## 剩余网络

### 定义

给定某个 st-流量网络和其 st-流量配置，这种配置下的剩余网络中的顶点和原网络相同。原网络中的每条边都对应着剩余网络中的 1~2 条边。它的定义如下：对于原网络中的每条从顶点 v 到 w 的边 e，令\$f_x\$ 表示它的流量，$c_e$表示它的容量。如果$f_e$为正，将`w->v`加入剩余网络且容量为$f_e$；如果$f_e$小于$c_e$，将 `w->w` 加入剩余网络且容量为$c_e-f_e$

## 网络流量中的边（剩余网络）

```java
public class FlowEdge {
    private final int v;
    private final int w;
    private final double capacity;
    private double flow;

    public FlowEdge(int v, int w, double capacity){
        ...
    }
    public int from(){return v;}
    public int to(){return w;}
    public double capacity(){return capacity;}
    public double flow(){return flow;}
    public int other(int vertex){
        return vertex == v ? w : v;
    }
    public double residualCapacityTo(int vertex){
        if (vertex == v) return flow;
        else if (vertex == w) return capacity -flow;
        else throw new RuntimeException("Inconsistent dege");
    }
    public void addResidualFlowTo(int vertex,double delta){
        if (vertex == v) flow -= delta;
        else if (vertex == w) flow += delta;
        else throw new RuntimeException("Inconsistent dege");
    }
}
```

## FordFulkerson 实现

```java
public class FordFulkerson{
    private boolean[] marked;
    private FlowEdge[] edgeTo;
    private double value;
    public FordFulkerson(FlowNetwork G,int s,int t){
         while (hasAugmentingPath(G, s, t)) {

            // 计算瓶颈
            double bottle = Double.POSITIVE_INFINITY;
            for (int v = t; v != s; v = edgeTo[v].other(v)) {
                bottle = Math.min(bottle, edgeTo[v].residualCapacityTo(v));
            }

            // 增大流量
            for (int v = t; v != s; v = edgeTo[v].other(v)) {
                edgeTo[v].addResidualFlowTo(v, bottle);
            }

            value += bottle;
        }

    }
    public double value()  {
        return value;
    }
    //返回true如果v属于最小切分
    public boolean inCut(int v)  {
        return marked[v];
    }

    private boolean hasAugmentingPath(FlowNetwork G, int s, int t) {
        edgeTo = new FlowEdge[G.V()];
        marked = new boolean[G.V()];

        // breadth-first search
        Queue<Integer> queue = new Queue<Integer>();
        queue.enqueue(s);
        marked[s] = true;
        while (!queue.isEmpty() && !marked[t]) {
            int v = queue.dequeue();

            for (FlowEdge e : G.adj(v)) {
                int w = e.other(v);

                // if residual capacity from v to w
                if (e.residualCapacityTo(w) > 0 && !marked[w]) {
                    edgeTo[w] = e;
                    marked[w] = true;
                    queue.enqueue(w);
                }
            }
        }

        // is there an augmenting path?
        return marked[t];
    }
}
```

