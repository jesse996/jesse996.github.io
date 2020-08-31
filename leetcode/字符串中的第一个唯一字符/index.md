# 字符串中的第一个唯一字符


第 387 题：字符串中的第一个唯一字符
给定一个字符串，找到它的第一个不重复的字符，并返回它的索引。如果不存在，则返回 -1 。  
案例:

```
s = "leetcode"
返回 0.
```

```
s = "loveleetcode",
返回 2.
```

注意事项： 您可以假定该字符串只包含小写字母。

java:

```java
public int firstUniqChar(String s) {
    int[] arr = new int[26];
    //先记录每个字符最后一次出现的引索
    for (int i = 0; i < s.length(); i++) {
        arr[s.charAt(i) - 'a'] = i;
    }
    //如果第一次引索和记录的不一样说明有重复
    for (int i = 0; i < s.length(); i++) {
        if (arr[s.charAt(i)-'a'] != i) {
            arr[s.charAt(i)-'a'] = -1;
        } else {
            return i;
        }
    }
    return -1;
}
```

rust:

```rust
pub fn first_uniq_char(s: String) -> i32 {
    let mut arr = [0i32; 26];
    for (i, c) in s.char_indices() {
        arr[c as usize - 'a' as usize] = i as i32;
    }
    for (i, c) in s.char_indices() {
        if i as i32 != arr[c as usize - 'a' as usize] {
            arr[c as usize - 'a' as usize] = -1;
        } else {
            return i as i32;
        }
    }
    -1
}
```

