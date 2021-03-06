# 两个数组的交集


## 第 350 题：两个数组的交集

给定两个数组，编写一个函数来计算它们的交集。

### 示例 1:

输入: `nums1 = [1,2,2,1]`, `nums2 = [2,2]`

输出: `[2,2]`

### 示例 2:

输入: `nums1 = [4,9,5], nums2 = [9,4,9,8,4]`

输出: `[4,9]`

说明：

-   输出结果中每个元素出现的次数，应与元素在两个数组中出现的次数一致。
-   我们可以不考虑输出结果的顺序。

进阶:

-   如果给定的数组已经排好序呢？将如何优化你的算法呢？

### 分析

> 首先拿到这道题，我们基本马上可以想到，此题可以看成是一道传统的映射题（map 映射），为什么可以这样看呢，因为我们需找出两个数组的交集元素，同时应与两个数组中出现的次数一致。这样就导致了我们需要知道每个值出现的次数，所以映射关系就成了<元素,出现次数>。剩下的就是顺利成章的解题。

```java
//方法一，用map
public int[] intersect(int[] nums1, int[] nums2) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i : nums1) {
        int count = map.getOrDefault(i, 0);
        map.put(i, count + 1);
    }
    int k = 0;
    for (int i : nums2) {
        int count = map.getOrDefault(i, 0);
        if (count > 0) {
            nums2[k++] = i;
            map.put(i, count - 1);
        }
    }
    return Arrays.copyOfRange(nums2, 0, k);
}
```

对于两个已经排序好数组的题，我们可以很容易想到使用双指针的解法:

> 如果两个指针的元素不相等，我们将小的一个指针后移,直到任意一个数组终止。

```java
//方法二：如果排好序
public int[] intersect2(int[] nums1, int[] nums2) {
    Arrays.sort(nums1);
    Arrays.sort(nums2);
    int i = 0, j = 0, k = 0;
    while (i < nums1.length && j < nums2.length) {
        if (nums1[i] == nums2[j]) {
            nums1[k] = nums1[i];
            i++;
            j++;
            k++;
        } else if (nums1[i] > nums2[j]) {
            j++;
        } else {
            i++;
        }
    }
    return Arrays.copyOfRange(nums1,0,k);
}
```

