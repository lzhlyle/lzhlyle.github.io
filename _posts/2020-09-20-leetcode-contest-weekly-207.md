---
layout:     post
title:      动态规划不可怕，思路不清想到炸。
subtitle:   LeetCode 第 207 场周赛
date:       2020-09-20
author:     lyle
header-img: "img/in-post/leetcode-contest-weekly-207/banner.jpeg"
tags:
    - LeetCode 竞赛
    - 回溯
    - 动态规划
---

> 12 分，300+ 名

## 题目

1. [重新排列单词间的空格](#t1-重新排列单词间的空格)
2. [拆分字符串使唯一子字符串的数目最大](#t2-拆分字符串使唯一子字符串的数目最大)
3. [矩阵的最大非负积](#t3-矩阵的最大非负积)
4. [连通两组点的最小成本](#t4-连通两组点的最小成本)

## T1. 重新排列单词间的空格

#### 题目

<div class="notranslate"><p>给你一个字符串 <code>text</code> ，该字符串由若干被空格包围的单词组成。每个单词由一个或者多个小写英文字母组成，并且两个单词之间至少存在一个空格。题目测试用例保证 <code>text</code> <strong>至少包含一个单词</strong> 。</p>

<p>请你重新排列空格，使每对相邻单词之间的空格数目都 <strong>相等</strong> ，并尽可能 <strong>最大化</strong> 该数目。如果不能重新平均分配所有空格，请 <strong>将多余的空格放置在字符串末尾</strong> ，这也意味着返回的字符串应当与原 <code>text</code> 字符串的长度相等。</p>

<p>返回 <strong>重新排列空格后的字符串</strong> 。</p>

<p><strong>示例 1：</strong></p>

<pre><strong>输入：</strong>text = "  this   is  a sentence "
<strong>输出：</strong>"this   is   a   sentence"
<strong>解释：</strong>总共有 9 个空格和 4 个单词。可以将 9 个空格平均分配到相邻单词之间，相邻单词间空格数为：9 / (4-1) = 3 个。
</pre>

<p><strong>示例 2：</strong></p>

<pre><strong>输入：</strong>text = " practice   makes   perfect"
<strong>输出：</strong>"practice   makes   perfect "
<strong>解释：</strong>总共有 7 个空格和 3 个单词。7 / (3-1) = 3 个空格加上 1 个多余的空格。多余的空格需要放在字符串的末尾。
</pre>

<p><strong>示例 3：</strong></p>

<pre><strong>输入：</strong>text = "hello   world"
<strong>输出：</strong>"hello   world"
</pre>

<p><strong>示例 4：</strong></p>

<pre><strong>输入：</strong>text = "  walks  udp package   into  bar a"
<strong>输出：</strong>"walks  udp  package  into  bar  a "
</pre>

<p><strong>示例 5：</strong></p>

<pre><strong>输入：</strong>text = "a"
<strong>输出：</strong>"a"
</pre>

<p><strong>提示：</strong></p>

<ul>
	<li><code>1 &lt;= text.length &lt;= 100</code></li>
	<li><code>text</code> 由小写英文字母和 <code>' '</code> 组成</li>
	<li><code>text</code> 中至少包含一个单词</li>
</ul>
</div>

#### 题解思路

- 提前算好间隔的空格数 `each`

#### 参考代码

```java
// bf
public String reorderSpaces(String text) {
    int blank = 0;
    for (char c : text.toCharArray())
        if (c == ' ') blank++;

    String[] arr = text.trim().split("\\s+");
    int word = arr.length;
    if (word == 1) {
        StringBuilder builder = new StringBuilder();
        builder.append(arr[0]);
        while (blank-- > 0)
            builder.append(" ");
        return builder.toString();
    }

    int each = blank / (word - 1);
    StringBuilder empty = new StringBuilder();
    for (int i = 0; i < each; i++)
        empty.append(" ");

    int rest = blank % (word - 1);
    StringBuilder restEmpty = new StringBuilder();
    for (int i = 0; i < rest; i++)
        restEmpty.append(" ");

    StringBuilder builder = new StringBuilder();
    for (int i = 0; i < word; i++) {
        builder.append(arr[i]);
        if (i != word - 1) builder.append(empty);
    }
    builder.append(restEmpty);
    return builder.toString();
}
```

#### 复杂度分析

- 时间复杂度：`O(n)`
- 空间复杂度：`O(n)`

## T2. 拆分字符串使唯一子字符串的数目最大

#### 题目

<div class="notranslate"><p>给你一个字符串 <code>s</code> ，请你拆分该字符串，并返回拆分后唯一子字符串的最大数目。</p>

<p>字符串 <code>s</code> 拆分后可以得到若干 <strong>非空子字符串</strong> ，这些子字符串连接后应当能够还原为原字符串。但是拆分出来的每个子字符串都必须是 <strong>唯一的</strong> 。</p>

<p>注意：<strong>子字符串</strong> 是字符串中的一个连续字符序列。</p>

<p><strong>示例 1：</strong></p>

<pre><strong>输入：</strong>s = "ababccc"
<strong>输出：</strong>5
<strong>解释：</strong>一种最大拆分方法为 ['a', 'b', 'ab', 'c', 'cc'] 。像 ['a', 'b', 'a', 'b', 'c', 'cc'] 这样拆分不满足题目要求，因为其中的 'a' 和 'b' 都出现了不止一次。
</pre>

<p><strong>示例 2：</strong></p>

<pre><strong>输入：</strong>s = "aba"
<strong>输出：</strong>2
<strong>解释：</strong>一种最大拆分方法为 ['a', 'ba'] 。
</pre>

<p><strong>示例 3：</strong></p>

<pre><strong>输入：</strong>s = "aa"
<strong>输出：</strong>1
<strong>解释：</strong>无法进一步拆分字符串。
</pre>

<p><strong>提示：</strong></p>

<ul>
	<li>
	<p><code>1 &lt;= s.length&nbsp;&lt;= 16</code></p>
	</li>
	<li>
	<p><code>s</code> 仅包含小写英文字母</p>
	</li>
</ul>
</div>

#### 题解思路

- 空间不大，回溯吧
- `HashSet` 记录是否出现过
    - 已有，必须等待后面的字母
    - 未有
        - 直接进 `set`
        - 不进，等待后面的字母
- 全局 `max` 求最大

#### 参考代码

```java
// bt
private int max;

public int maxUniqueSplit(String s) {
    max = 0;
    dfs(s.toCharArray(), 0, "", new HashSet<>());
    return max;
}

private void dfs(char[] arr, int i, String curr, Set<String> set) {
    if (i == arr.length) {
        max = Math.max(max, set.size());
        return;
    }

    char c = arr[i];
    String comb = curr + c;
    if (set.contains(comb)) {
        // 已有，必须再等
        dfs(arr, i + 1, comb, set);
    } else {
        // 未有
        // 选择不等了直接进
        set.add(comb);
        dfs(arr, i + 1, "", set);
        set.remove(comb);

        // 选择继续等待
        dfs(arr, i + 1, comb, set);
    }
}
```

#### 复杂度分析

- 时间复杂度：`2^n`
    - `T(n) = 2T(n - 1) + O(1)`
- 空间复杂度：`O(n)`

## T3. 矩阵的最大非负积

#### 题目

<div class="notranslate"><p>给你一个大小为 <code>rows x cols</code> 的矩阵 <code>grid</code> 。最初，你位于左上角 <code>(0, 0)</code> ，每一步，你可以在矩阵中 <strong>向右</strong> 或 <strong>向下</strong> 移动。</p>

<p>在从左上角 <code>(0, 0)</code> 开始到右下角 <code>(rows - 1, cols - 1)</code> 结束的所有路径中，找出具有 <strong>最大非负积</strong> 的路径。路径的积是沿路径访问的单元格中所有整数的乘积。</p>

<p>返回 <strong>最大非负积 </strong>对<strong><em> </em><code>10<sup>9</sup>&nbsp;+ 7</code></strong> <strong>取余</strong> 的结果。如果最大积为负数，则返回<em> </em><code>-1</code> 。</p>

<p><strong>注意，</strong>取余是在得到最大积之后执行的。</p>

<p><strong>示例 1：</strong></p>

<pre><strong>输入：</strong>grid = [[-1,-2,-3],
&nbsp;            [-2,-3,-3],
&nbsp;            [-3,-3,-2]]
<strong>输出：</strong>-1
<strong>解释：</strong>从 (0, 0) 到 (2, 2) 的路径中无法得到非负积，所以返回 -1
</pre>

<p><strong>示例 2：</strong></p>

<pre><strong>输入：</strong>grid = [[<strong>1</strong>,-2,1],
&nbsp;            [<strong>1</strong>,<strong>-2</strong>,1],
&nbsp;            [3,<strong>-4</strong>,<strong>1</strong>]]
<strong>输出：</strong>8
<strong>解释：</strong>最大非负积对应的路径已经用粗体标出 (1 * 1 * -2 * -4 * 1 = 8)
</pre>

<p><strong>示例 3：</strong></p>

<pre><strong>输入：</strong>grid = [[<strong>1</strong>, 3],
&nbsp;            [<strong>0</strong>,<strong>-4</strong>]]
<strong>输出：</strong>0
<strong>解释：</strong>最大非负积对应的路径已经用粗体标出 (1 * 0 * -4 = 0)
</pre>

<p><strong>示例 4：</strong></p>

<pre><strong>输入：</strong>grid = [[ <strong>1</strong>, 4,4,0],
&nbsp;            [<strong>-2</strong>, 0,0,1],
&nbsp;            [ <strong>1</strong>,<strong>-1</strong>,<strong>1</strong>,<strong>1</strong>]]
<strong>输出：</strong>2
<strong>解释：</strong>最大非负积对应的路径已经用粗体标出 (1 * -2 * 1 * -1 * 1 * 1 = 2)
</pre>

<p><strong>提示：</strong></p>

<ul>
	<li><code>1 &lt;= rows, cols &lt;= 15</code></li>
	<li><code>-4 &lt;= grid[i][j] &lt;= 4</code></li>
</ul>
</div>

#### 题解思路

- 显然 DP
- `dp[i][j][0|1]` 表示走到 `g[i][j]` 时的最小值 `dp[i][j][0]`、最大值 `dp[i][j][1]`

#### 参考代码

```java
// dp
private static final int MOD = (int) 1e9 + 7;

public int maxProductPath(int[][] g) {
    int m = g.length, n = g[0].length;
    long[][][] dp = new long[m][n][2]; // 2: {min, max}
    dp[0][0][0] = dp[0][0][1] = (long) g[0][0];

    for (int i = 1; i < m; i++)
        dp[i][0][0] = dp[i][0][1] = dp[i - 1][0][1] * g[i][0];
    for (int j = 1; j < n; j++)
        dp[0][j][0] = dp[0][j][1] = dp[0][j - 1][1] * g[0][j];

    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            long u1 = dp[i - 1][j][0] * g[i][j];
            long u2 = dp[i - 1][j][1] * g[i][j];
            long l1 = dp[i][j - 1][0] * g[i][j];
            long l2 = dp[i][j - 1][1] * g[i][j];
            dp[i][j][0] = Math.min(u1, Math.min(u2, Math.min(l1, l2)));
            dp[i][j][1] = Math.max(u1, Math.max(u2, Math.max(l1, l2)));
        }
    }
    int res = (int) (dp[m - 1][n - 1][1] % MOD);
    return res < 0 ? -1 : res;
}
```

#### 复杂度分析

- 时间复杂度：`O(mn)`
- 空间复杂度：`O(mn)`

## T4. 连通两组点的最小成本

#### 题目

<div class="notranslate"><p>给你两组点，其中第一组中有 <code>size<sub>1</sub></code> 个点，第二组中有 <code>size<sub>2</sub></code> 个点，且 <code>size<sub>1</sub> &gt;= size<sub>2</sub></code> 。</p>

<p>任意两点间的连接成本 <code>cost</code> 由大小为 <code>size<sub>1</sub> x size<sub>2</sub></code> 矩阵给出，其中 <code>cost[i][j]</code> 是第一组中的点 <code>i</code> 和第二组中的点 <code>j</code> 的连接成本。<strong>如果两个组中的每个点都与另一组中的一个或多个点连接，则称这两组点是连通的。</strong>换言之，第一组中的每个点必须至少与第二组中的一个点连接，且第二组中的每个点必须至少与第一组中的一个点连接。</p>

<p>返回连通两组点所需的最小成本。</p>

<p><strong>示例 1：</strong></p>

<p><img style="height: 243px; width: 322px;" src="/img/in-post/leetcode-contest-weekly-207/t4-eg1.png" alt=""></p>

<pre><strong>输入：</strong>cost = [[15, 96], [36, 2]]
<strong>输出：</strong>17
<strong>解释：</strong>连通两组点的最佳方法是：
1--A
2--B
总成本为 17 。
</pre>

<p><strong>示例 2：</strong></p>

<p><img style="height: 403px; width: 322px;" src="/img/in-post/leetcode-contest-weekly-207/t4-eg2.png" alt=""></p>

<pre><strong>输入：</strong>cost = [[1, 3, 5], [4, 1, 1], [1, 5, 3]]
<strong>输出：</strong>4
<strong>解释：</strong>连通两组点的最佳方法是：
1--A
2--B
2--C
3--A
最小成本为 4 。
请注意，虽然有多个点连接到第一组中的点 2 和第二组中的点 A ，但由于题目并不限制连接点的数目，所以只需要关心最低总成本。</pre>

<p><strong>示例 3：</strong></p>

<pre><strong>输入：</strong>cost = [[2, 5, 1], [3, 4, 7], [8, 1, 2], [6, 2, 4], [3, 8, 8]]
<strong>输出：</strong>10
</pre>

<p><strong>提示：</strong></p>

<ul>
	<li><code>size<sub>1</sub> == cost.length</code></li>
	<li><code>size<sub>2</sub> == cost[i].length</code></li>
	<li><code>1 &lt;= size<sub>1</sub>, size<sub>2</sub> &lt;= 12</code></li>
	<li><code>size<sub>1</sub> &gt;=&nbsp;size<sub>2</sub></code></li>
	<li><code>0 &lt;= cost[i][j] &lt;= 100</code></li>
</ul>
</div>

#### 题解思路

- [花花酱 LeetCode 1595. Minimum Cost to Connect Two Groups of Points](https://www.bilibili.com/video/BV1Xf4y1D7SW)
- `dp[i][k]` 表示左侧已完成共 `i` 个点（不是第 `i`），右侧连接情况为 `k`（位运算表示）
- 对右侧第 `j` 个点，`dp[i][k | (1 << j)]` 可来自
    - `dp[i][k]` 左侧已连的，对应右侧未连的 `j`
    - `dp[i - 1][k]` 左侧未连的，对应右侧未连的 `j`
    - 至于左侧已连/未连是谁，枚举 `i` 即可

#### 参考代码

```java
public int connectTwoGroups(List<List<Integer>> cost) {
    int l = cost.size(), r = cost.get(0).size();

    int[][] dp = new int[l + 1][1 << r];
    for (int[] d : dp) Arrays.fill(d, (int) 1e9);
    dp[0][0] = 0;

    for (int i = 0; i < l; i++)
        for (int j = 0; j < r; j++)
            for (int k = 0; k < (1 << r); k++)
                dp[i + 1][k | (1 << j)] = 
                    Math.min(
                        dp[i + 1][k | (1 << j)],
                        Math.min(
                            dp[i][k] + cost.get(i).get(j)
                            dp[i + 1][k] + cost.get(i).get(j),
                        )
                    );
    return dp[l][(1 << r) - 1];
}
```

#### 复杂度分析

- 时间复杂度：`O(mn * 2^n)`
- 空间复杂度：`O(m * 2^n)`
