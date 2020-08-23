---
layout:     post
title:      专项与坚持，做时间的朋友
subtitle:   LeetCode 第 33 场双周赛
date:       2020-08-22
author:     lyle
header-img: "img/in-post/leetcode-contest-biweekly-33/banner.jpg"
tags:
    - LeetCode 竞赛
    - 
---

难得 1 小时内 AK，难度不大，排名需要细心与手速。

> 18 分，250+ 名

## 题目

1. [千位分隔数](#t1-千位分隔数)
2. [可以到达所有点的最少点数目](#t2-可以到达所有点的最少点数目)
3. [得到目标数组的最少函数调用次数](#t3-得到目标数组的最少函数调用次数)
4. [二维网格图中探测环](#t4-二维网格图中探测环)
5. [赛后复盘](#赛后复盘)

## T1. 千位分隔数

#### 题目

> 给你一个整数 `n`，请你每隔三位添加点（即 "." 符号）作为千位分隔符，并将结果以字符串格式返回。

示例 1：

```
输入：n = 987
输出："987"
```

示例 2：

```
输入：n = 1234
输出："1.234"
```

示例 3：

```
输入：n = 123456789
输出："123.456.789"
```

示例 4：

```
输入：n = 0
输出："0"
```

提示：

- `0 <= n < 2^31`

#### 题解思路

- 倒序分割，组装字符串，再倒序输出

#### 参考代码

```java
public String thousandSeparator(int n) {
    if (n < 1000) return String.valueOf(n);
    char[] arr = String.valueOf(n).toCharArray();
    StringBuilder builder = new StringBuilder();
    for (int i = arr.length - 1, cnt = 1; i >= 0; i--, cnt++) {
        if (cnt % 3 == 1) builder.append(".");
        builder.append(arr[i]);
    }
    return builder.reverse().toString().substring(0, builder.length() - 1);
}
```

#### 复杂度分析

- 时间复杂度：`O(n)`
- 空间复杂度：`O(n)`，可通过计算省掉 `arr` 的占用达到 `O(1)`

## T2. 可以到达所有点的最少点数目

#### 题目

> 给你一个 **有向无环图**， `n` 个节点编号为 `0` 到 `n-1` ，以及一个边数组 `edges`，其中 `edges[i] = [fromi, toi]` 表示一条从点 `fromi` 到点 `toi` 的有向边。
> 
> 找到最小的点集使得从这些点出发能到达图中所有点。题目保证解存在且唯一。
> 
> 你可以以任意顺序返回这些节点编号。

示例 1：

![t2 eg1](/img/in-post/leetcode-contest-biweekly-33/t2-eg1.png)

```
输入：n = 6, edges = [[0,1],[0,2],[2,5],[3,4],[4,2]]
输出：[0,3]
解释：从单个节点出发无法到达所有节点。从 0 出发我们可以到达 [0,1,2,5] 。从 3 出发我们可以到达 [3,4,2,5] 。所以我们输出 [0,3] 。
```

示例 2：

![t2 eg2](/img/in-post/leetcode-contest-biweekly-33/t2-eg2.png)

```
输入：n = 5, edges = [[0,1],[2,1],[3,1],[1,4],[2,4]]
输出：[0,2,3]
解释：注意到节点 0，3 和 2 无法从其他节点到达，所以我们必须将它们包含在结果点集中，这些点都能到达节点 1 和 4 。
```

提示：

- `2 <= n <= 10^5`
- `1 <= edges.length <= min(10^5, n * (n - 1) / 2)`
- `edges[i].length == 2`
- `0 <= fromi, toi < n`
- 所有点对 `(fromi, toi)` 互不相同。

#### 题解思路

- 无入度的点

#### 参考代码

```java
public List<Integer> findSmallestSetOfVertices(int n, List<List<Integer>> edges) {
    Set<Integer> set = new HashSet<>();
    for (int i = 0; i < n; i++)
        set.add(i);
    for (List<Integer> e : edges)
        set.remove(e.get(1));
    return new ArrayList<>(set);
}
```

#### 复杂度分析

- 时间复杂度：`O(n)`
- 空间复杂度：`O(n)`

## T3. 得到目标数组的最少函数调用次数

#### 题目

![t3](/img/in-post/leetcode-contest-biweekly-33/t3.png)

> 给你一个与 `nums` 大小相同且初始值全为 `0` 的数组 `arr` ，请你调用以上函数得到整数数组 `nums`。
> 
> 请你返回将 `arr` 变成 `nums` 的最少函数调用次数。
> 
> 答案保证在 32 位有符号整数以内。

示例 1：

```
输入：nums = [1,5]
输出：5
解释：给第二个数加 1 ：[0, 0] 变成 [0, 1] （1 次操作）。
将所有数字乘以 2 ：[0, 1] -> [0, 2] -> [0, 4] （2 次操作）。
给两个数字都加 1 ：[0, 4] -> [1, 4] -> [1, 5] （2 次操作）。
总操作次数为：1 + 2 + 2 = 5 。
```

示例 2：

```
输入：nums = [2,2]
输出：3
解释：给两个数字都加 1 ：[0, 0] -> [0, 1] -> [1, 1] （2 次操作）。
将所有数字乘以 2 ： [1, 1] -> [2, 2] （1 次操作）。
总操作次数为： 2 + 1 = 3 。
```

示例 3：

```
输入：nums = [4,2,5]
输出：6
解释：（初始）[0,0,0] -> [1,0,0] -> [1,0,1] -> [2,0,2] -> [2,1,2] -> [4,2,4] -> [4,2,5] （nums 数组）。
```

示例 4：

```
输入：nums = [3,2,2,4]
输出：7
```

示例 5：

```
输入：nums = [2,4,8,16]
输出：8
```

提示：

- `1 <= nums.length <= 10^5`
- `0 <= nums[i] <= 10^9`

#### 题解思路 1

- 题意：指定数 +1 或 所有数翻倍
- 反推：指定数 -1 或 所有数减半
- 暴力模拟

#### 参考代码 1

```java
public int minOperations(int[] nums) {
    int cnt = 0, zero = 0, move = 0;
    for (int i = 0; i < nums.length; i++) {
        if (nums[i] == 0) zero++;
        else {
            cnt += nums[i] & 1;
            if (nums[i] > 1) {
                nums[i] >>>= 1;
                move = 1;
            } else nums[i] = 0;
        }
    }
    if (zero == nums.length) return 0;
    return cnt + move + minOperations(nums);
}
```

#### 复杂度分析 1

- 时间复杂度：`O(nlog(max))`，`T(n) = T(n / 2) + O(n)`
- 空间复杂度：`O(1)`

#### 题解思路 2

- 所有数看作二进制表示
- 所有数逐步右移
- 所有低位 `1` 都需要操作一次变成 `0`，且 **操作后数位不变**
- 右移最多次数为最长的二进制表示数

#### 参考代码 2

```java
public int minOperations(int[] nums) {
    int cnt = 0, max = nums[0];
    
    // 共多少个 1
    for (int v : nums) {
        max = Math.max(max, v);
        while (v != 0) {
            cnt++;
            v &= v - 1;
        }
    }
    
    // 最大多少位
    while (max != 0) {
        cnt++;
        max >>>= 1;
    }
    return cnt - 1;
}
```

#### 复杂度分析 2

- 时间复杂度：`O(32n)`
- 空间复杂度：`O(1)`

## T4. 二维网格图中探测环

#### 题目

> 给你一个二维字符网格数组 `grid`，大小为 `m x n`，你需要检查 `grid` 中是否存在 **相同值** 形成的环。
> 
> 一个环是一条开始和结束于同一个格子的长度 **大于等于 `4`** 的路径。对于一个给定的格子，你可以移动到它上、下、左、右四个方向相邻的格子之一，可以移动的前提是这两个格子有 **相同的值**。
> 
> 同时，你也不能回到上一次移动时所在的格子。比方说，环 `(1, 1) -> (1, 2) -> (1, 1)` 是不合法的，因为从 `(1, 2)` 移动到 `(1, 1)` 回到了上一次移动时的格子。
> 
> 如果 `grid` 中有相同值形成的环，请你返回 `true`，否则返回 `false`。

示例 1：

![t4 eg1 1](/img/in-post/leetcode-contest-biweekly-33/t4-eg1-1.png)

![t4 eg1 2](/img/in-post/leetcode-contest-biweekly-33/t4-eg1-2.png)

```
输入：grid = [["a","a","a","a"],["a","b","b","a"],["a","b","b","a"],["a","a","a","a"]]
输出：true
解释：如下图所示，有 2 个用不同颜色标出来的环：
```

示例 2：

![t4 eg2 1](/img/in-post/leetcode-contest-biweekly-33/t4-eg2-1.png)

![t4 eg2 2](/img/in-post/leetcode-contest-biweekly-33/t4-eg2-2.png)

```
输入：grid = [["c","c","c","a"],["c","d","c","c"],["c","c","e","c"],["f","c","c","c"]]
输出：true
解释：如下图所示，只有高亮所示的一个合法环：
```

示例 3：

![t4 eg3](/img/in-post/leetcode-contest-biweekly-33/t4-eg3.png)

```
输入：grid = [["a","b","b"],["b","z","b"],["b","b","a"]]
输出：false
```

提示：

- `m == grid.length`
- `n == grid[i].length`
- `1 <= m <= 500`
- `1 <= n <= 500`
- `grid` 只包含小写英文字母。

#### 题解思路

- 深度优先，四连通图
- 访问记录 `visited: int[m][n]` 存储走到当前的步数（即距离 `dis`），避免来回走

#### 参考代码

```java
private static final int[] dx = {-1, 0, 1, 0}, dy = {0, -1, 0, 1};
private char[][] mat;
private int m, n;
private int[][] visited;

public boolean containsCycle(char[][] grid) {
    mat = grid;
    m = mat.length;
    n = mat[0].length;
    visited = new int[m][n];
    if (m == 1 || n == 1) return false;

    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            if (dfs(i, j, 1)) return true;
    return false;
}

private boolean dfs(int i, int j, int dis) {
    // 判断是否成环
    if (visited[i][j] > 0) return dis - visited[i][j] > 2;
    
    visited[i][j] = dis; // 期望的第 dis 步
    for (int di = 0; di < 4; di++) {
        int x = i + dx[di], y = j + dy[di];
        if (x >= 0 && x < m && y >= 0 && y < n)
            if (mat[x][y] == mat[i][j])
                if (dfs(x, y, dis + 1)) return true;
    }
    return false;
}
```

#### 复杂度分析

- 时间复杂度：`O(mn)`
- 空间复杂度：`O(mn)`

## 赛后复盘

- T1 纯字符串玩法，怎么都能过，竞赛就不写那么精简了 XD
- T2 没有套路，直来直往，理解图的基本算法就很轻松
- T3 感谢官方对暴力没有卡，`O(31 * 10 ^ 5)` 还是挺危险的，位运算着实没想到，赛后看零神实况才明白，写起来要调一下，尤其注意 **`1` 变 `0` 操作后数位不变**
- T4 深度优先的基础上稍微要求了不要来回走，但要判断「绕回来」的环，也没什么套路。赛中提交错了一次，错在 `boolean[][] visited` 未记步数，导致来回走没能判断好

很开心慢慢可以 AK 了！

从首次参赛只能做一两题，到保二争三，再到保三争四，简单一些的竞赛可以迅速三题之后有希望 AK。感受到自己在进步，也是非常激励的事情。

专项、反复练习，收效会一点一点到来：做时间的朋友。

