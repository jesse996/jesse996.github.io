# 数独游戏


# 数独游戏

今天看到一个比较游戏以的题目，就是数独游戏。
先说一下数独的规则：

1. 数字 1-9 在每一行只能出现一次。
2. 数字 1-9 在每一列只能出现一次。
3. 数字 1-9 在每一个以粗实线分隔的 3x3 宫内只能出现一次。

如：
![数独](http://upload.wikimedia.org/wikipedia/commons/thumb/f/ff/Sudoku-by-L2G-20050714.svg/250px-Sudoku-by-L2G-20050714.svg.png)

一个解：
![解](http://upload.wikimedia.org/wikipedia/commons/thumb/3/31/Sudoku-by-L2G-20050714_solution.svg/250px-Sudoku-by-L2G-20050714_solution.svg.png)

题目给的形式如下：

```java
    public void solveSudoku(char[][] board) {
    }
```

有如下规定：

- 给定的数独序列只包含数字 1-9 和字符 '.' 。
- 你可以假设给定的数独只有唯一解。
- 给定数独永远是 9x9 形式的。

首先看到题目，只需求一个解就行了，像这种数独游戏无非就是暴力寻找，立马可以想到用回溯法。

```java
    public void solveSudoku(char[][] board) {
        backtrace(board, 0, 0);
    }

    //从r行c列开始往下找是否有解
    public boolean backtrace(char[][] board, int r, int c) {
        //找到解，直接返回
        if (r == 9) return true;
        //到下一行
        if (c == 9) {
            return backtrace(board, r + 1, 0);
        }
        //选择
        for (char k = '1'; k <= '9'; k++) {
            //有数字，就直接往下一个
            if (board[r][c] != '.') {
                return backtrace(board, r, c + 1);
            }
            if (!valid(board, r, c, k)) continue;
            //做选择
            board[r][c] = k;
            if (backtrace(board, r, c + 1)) {
                return true;
            }
            //取消选择
            board[r][c] = '.';
        }
        return false;
    }
```

`valid`函数就是看适合合法了：

```java
    public boolean valid(char[][] board, int r, int c, int ch) {
        for (int i = 0; i < 9; i++) {
            //看当前列是否合法
            if (board[i][c] == ch) return false;
            //看当前行是否合法
            if (board[r][i] == ch) return false;
            //看周围 3x3 宫内是否合法
            if (board[(r / 3) * 3 + i / 3][(c / 3) * 3 + i % 3] == ch) return false;
        }
        return true;
    }
```

好了，这就是全部了～题目虽然不是很难，但小时候玩过的游戏，现在用程序解决有不一样的乐趣～

