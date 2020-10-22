# 32-1


# 从上到下打印二叉树

从上到下打印出二叉树的每个节点，同一层的节点按照从左到右的顺序打印。

例如:

给定二叉树: [3,9,20,null,null,15,7],

```
    3
   / \
  9  20
    /  \
   15   7
```

返回：

[3,9,20,15,7]

---

来源：力扣（LeetCode）  
链接：<https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof>  
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

---

题目要求从上到下，从左到右打印，自然就想到用队列来循环。也就是 bfs 算法。

```java
class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;

    TreeNode(int x) {
        val = x;
    }
}
public int[] levelOrder(TreeNode root) {
    if (root == null) return new int[0];
    Queue<TreeNode> queue = new ArrayDeque<>() {{
        add(root);
    }};
    List<Integer> list = new ArrayList<>();
    while (!queue.isEmpty()) {
        TreeNode node = queue.poll();
        list.add(node.val);
        if (node.left != null) queue.add(node.left);
        if (node.right != null) queue.add(node.right);
    }
    int[] ret = new int[list.size()];
    for (int i = 0; i < list.size(); i++) {
        ret[i] = list.get(i);
    }
    return ret;
}
```

```rust
// Definition for a binary tree node.
#[derive(Debug, PartialEq, Eq)]
pub struct TreeNode {
    pub val: i32,
    pub left: Option<Rc<RefCell<TreeNode>>>,
    pub right: Option<Rc<RefCell<TreeNode>>>,
}

impl TreeNode {
    #[inline]
    pub fn new(val: i32) -> Self {
        TreeNode {
            val,
            left: None,
            right: None,
        }
    }
}

use std::rc::Rc;
use std::cell::RefCell;
use std::collections::VecDeque;


pub fn level_order(root: Option<Rc<RefCell<TreeNode>>>) -> Vec<i32> {
    if root.is_none() { return vec![]; }
    let root = root.unwrap();
    let mut queue = VecDeque::new();
    queue.push_back(root.clone());
    let mut ret = vec![];
    while !queue.is_empty() {
        let node = queue.pop_front().unwrap();
        ret.push(node.borrow().val);
        if node.borrow().left.is_some() {
            let left = Rc::clone( node.borrow().left.as_ref().unwrap());
            queue.push_back(left);
        }
        if node.borrow().right.is_some() {
            let left = Rc::clone( node.borrow().right.as_ref().unwrap());
            queue.push_back(left);
        }
    }
    ret
}
```

