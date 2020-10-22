# 最长公共前缀


## 题目 14: 最长公共前缀

编写一个函数来查找字符串数组中的最长公共前缀。如果不存在公共前缀，则返回""

### 示例 1:

输入: `["flower","flow","flight"]`  
输出: `"fl"`

### 示例 2:

输入: `["dog","racecar","car"]`  
输出: `""`  
解释:输入不存在公共前缀。

#### 说明：

_所有输入只包含小写字母 a-z_

## 分析

> 我们要想寻找最长公共前缀，那么首先这个前缀是公共的，我们可以从任意一个元素中找到它。那么首先，我们可以将第一个元素设置为基准元素 x0。假如数组为`["flow","flower","flight"]`，flow 就是我们的基准元素 x0。

> 然后我们只需要依次将基准元素和后面的元素进行比较（假定后面的元素依次为 `x1,x2,x3....`），不断更新基准元素(截取掉基准元素最后一个元素)，直到基准元素和所有元素都满足最长公共前缀的条件，就可以得到最长公共前缀。

```java
public String longestCommonPrefix(String[] strs) {
    if (strs.length == 0) return "";
    String prefix = strs[0];
    for (String str : strs) {
        while (str.indexOf(prefix) != 0) {
            if (prefix.length() <= 0) return "";
            prefix = prefix.substring(0, prefix.length() - 1);
        }
    }
    return prefix;
}
```

```rust
pub fn longest_common_prefix(strs: Vec<String>) -> String {
    if strs.is_empty() { return "".to_string(); }
    let mut prefix = &strs[0][..];
    for str in strs.iter() {
        while !str.starts_with(prefix) {
            if prefix.len() == 0 { return "".to_string(); }
            prefix = &prefix[0..prefix.len() - 1];
        }
    }
    prefix.into()
}
```

