---
layout:     post
title:      果然：没有「本可以」，也就没有遗憾
subtitle:   LeetCode 第 206 场周赛
date:       2020-09-13
author:     lyle
header-img: "img/in-post/leetcode-contest-weekly-206/banner.jpg"
tags:
    - LeetCode 竞赛
    - 优先级队列
    - 并查集
---

三题，知足了。

> 12 分，600+ 名

## 题目

1. [二进制矩阵中的特殊位置](#t1-二进制矩阵中的特殊位置)
2. [统计不开心的朋友](#t2-统计不开心的朋友)
3. [连接所有点的最小费用](#t3-连接所有点的最小费用)
4. [检查字符串是否可以通过排序子字符串得到另一个字符串](#t4-检查字符串是否可以通过排序子字符串得到另一个字符串)
5. [赛后复盘](#赛后复盘)

## T1. 二进制矩阵中的特殊位置

#### 题目

<div class="notranslate"><p>给你一个大小为 <code>rows x cols</code> 的矩阵 <code>mat</code>，其中 <code>mat[i][j]</code> 是 <code>0</code> 或 <code>1</code>，请返回 <strong>矩阵&nbsp;<em><code>mat</code></em> 中特殊位置的数目</strong> 。</p>

<p><strong>特殊位置</strong> 定义：如果 <code>mat[i][j] == 1</code> 并且第 <code>i</code> 行和第 <code>j</code> 列中的所有其他元素均为 <code>0</code>（行和列的下标均 <strong>从 0 开始</strong> ），则位置 <code>(i, j)</code> 被称为特殊位置。</p>

<p><strong>示例 1：</strong></p>

<pre><strong>输入：</strong>mat = [[1,0,0],
&nbsp;           [0,0,<strong>1</strong>],
&nbsp;           [1,0,0]]
<strong>输出：</strong>1
<strong>解释：</strong>(1,2) 是一个特殊位置，因为 mat[1][2] == 1 且所处的行和列上所有其他元素都是 0
</pre>

<p><strong>示例 2：</strong></p>

<pre><strong>输入：</strong>mat = [[<strong>1</strong>,0,0],
&nbsp;           [0,<strong>1</strong>,0],
&nbsp;           [0,0,<strong>1</strong>]]
<strong>输出：</strong>3
<strong>解释：</strong>(0,0), (1,1) 和 (2,2) 都是特殊位置
</pre>

<p><strong>示例 3：</strong></p>

<pre><strong>输入：</strong>mat = [[0,0,0,<strong>1</strong>],
&nbsp;           [<strong>1</strong>,0,0,0],
&nbsp;           [0,1,1,0],
&nbsp;           [0,0,0,0]]
<strong>输出：</strong>2
</pre>

<p><strong>示例 4：</strong></p>

<pre><strong>输入：</strong>mat = [[0,0,0,0,0],
&nbsp;           [<strong>1</strong>,0,0,0,0],
&nbsp;           [0,<strong>1</strong>,0,0,0],
&nbsp;           [0,0,<strong>1</strong>,0,0],
&nbsp;           [0,0,0,1,1]]
<strong>输出：</strong>3
</pre>

<p><strong>提示：</strong></p>

<ul>
	<li><code>rows == mat.length</code></li>
	<li><code>cols == mat[i].length</code></li>
	<li><code>1 &lt;= rows, cols &lt;= 100</code></li>
	<li><code>mat[i][j]</code> 是 <code>0</code> 或 <code>1</code></li>
</ul>
</div>

#### 题解思路

- 遍历行，记录这行里的哪些列可能入选
- 遍历候选列，统计结果
- 其他思路
    - 三层循环
    - 深度优先

#### 参考代码

```java
public int numSpecial(int[][] mat) {
    List<Integer> cols = new ArrayList<>();
    for (int[] row : mat) {
        int cnt = 0, colIndex = -1;
        for (int j = 0; j < mat[0].length && cnt <= 1; j++) {
            if (row[j] == 1) {
                cnt++;
                colIndex = j;
            }
        }
        if (cnt == 1) cols.add(colIndex);
    }

    int res = 0;
    for (int j : cols) {
        int cnt = 0;
        for (int[] row : mat) {
            if (row[j] == 1) cnt++;
        }
        if (cnt == 1) res++;
    }
    return res;
}
```

#### 复杂度分析

- 时间复杂度：`O(rows * cols)`
- 空间复杂度：`O(cols)`

## T2. 统计不开心的朋友

#### 题目

<div class="notranslate"><p>给你一份 <code>n</code> 位朋友的亲近程度列表，其中 <code>n</code> 总是 <strong>偶数</strong> 。</p>

<p>对每位朋友 <code>i</code>，<code>preferences[i]</code> 包含一份 <strong>按亲近程度从高</strong><strong>到低排列</strong> 的朋友列表。换句话说，排在列表前面的朋友与 <code>i</code> 的亲近程度比排在列表后面的朋友更高。每个列表中的朋友均以 <code>0</code> 到 <code>n-1</code> 之间的整数表示。</p>

<p>所有的朋友被分成几对，配对情况以列表 <code>pairs</code> 给出，其中 <code>pairs[i] = [x<sub>i</sub>, y<sub>i</sub>]</code> 表示 <code>x<sub>i</sub></code> 与 <code>y<sub>i</sub></code> 配对，且 <code>y<sub>i</sub></code> 与 <code>x<sub>i</sub></code> 配对。</p>

<p>但是，这样的配对情况可能会是其中部分朋友感到不开心。在 <code>x</code> 与 <code>y</code> 配对且 <code>u</code> 与 <code>v</code> 配对的情况下，如果同时满足下述两个条件，<code>x</code> 就会不开心：</p>

<ul>
	<li><code>x</code> 与 <code>u</code> 的亲近程度胜过 <code>x</code> 与 <code>y</code>，且</li>
	<li><code>u</code> 与 <code>x</code> 的亲近程度胜过 <code>u</code> 与 <code>v</code></li>
</ul>

<p>返回 <strong>不开心的朋友的数目</strong> 。</p>

<p><strong>示例 1：</strong></p>

<pre><strong>输入：</strong>n = 4, preferences = [[1, 2, 3], [3, 2, 0], [3, 1, 0], [1, 2, 0]], pairs = [[0, 1], [2, 3]]
<strong>输出：</strong>2
<strong>解释：</strong>
朋友 1 不开心，因为：
- <strong>1 与 0 </strong>配对，但 <strong>1 与 3</strong> 的亲近程度比 <strong>1 与 0</strong> 高，且
- <strong>3 与 1</strong> 的亲近程度比 <strong>3 与 2</strong> 高。
朋友 3 不开心，因为：
- 3 与 2 配对，但 <strong>3 与 1</strong> 的亲近程度比 <strong>3 与 2</strong> 高，且
- <strong>1 与 3</strong> 的亲近程度比 <strong>1 与 0</strong> 高。
朋友 0 和 2 都是开心的。
</pre>

<p><strong>示例 2：</strong></p>

<pre><strong>输入：</strong>n = 2, preferences = [[1], [0]], pairs = [[1, 0]]
<strong>输出：</strong>0
<strong>解释：</strong>朋友 0 和 1 都开心。
</pre>

<p><strong>示例 3：</strong></p>

<pre><strong>输入：</strong>n = 4, preferences = [[1, 3, 2], [2, 3, 0], [1, 3, 0], [0, 2, 1]], pairs = [[1, 3], [0, 2]]
<strong>输出：</strong>4
</pre>

<p><strong>提示：</strong></p>

<ul>
	<li><code>2 &lt;= n &lt;= 500</code></li>
	<li><code>n</code> 是偶数</li>
	<li><code>preferences.length&nbsp;== n</code></li>
	<li><code>preferences[i].length&nbsp;== n - 1</code></li>
	<li><code>0 &lt;= preferences[i][j] &lt;= n - 1</code></li>
	<li><code>preferences[i]</code> 不包含 <code>i</code></li>
	<li><code>preferences[i]</code> 中的所有值都是独一无二的</li>
	<li><code>pairs.length&nbsp;== n/2</code></li>
	<li><code>pairs[i].length&nbsp;== 2</code></li>
	<li><code>x<sub>i</sub> != y<sub>i</sub></code></li>
	<li><code>0 &lt;= x<sub>i</sub>, y<sub>i</sub>&nbsp;&lt;= n - 1</code></li>
	<li>每位朋友都 <strong>恰好</strong> 被包含在一对中</li>
</ul>
</div>

#### 题解思路

- 语文题~
- 提前准备哈希映射
- 注意别算重复了

#### 参考代码

```java
public int unhappyFriends(int n, int[][] preferences, int[][] pairs) {
    Map<Integer, Map<Integer, Integer>> pre = new HashMap<>(); // (x, (each, index))
    for (int i = 0; i < n; i++) {
        Map<Integer, Integer> pp = new HashMap<>();
        for (int j = 0; j < n - 1; j++) {
            pp.put(preferences[i][j], j);
        }
        pre.put(i, pp);
    }

    Map<Integer, Integer> map = new HashMap<>();
    for (int[] p : pairs) {
        map.put(p[0], p[1]);
        map.put(p[1], p[0]);
    }

    int cnt = 0;
    for (int x = 0; x < n; x++) {
        int y = map.get(x);
        for (int u = 0; u < n; u++) {
            if (u == x || u == y) continue;
            int v = map.get(u);

            // 对 x
            // 在 x 中：u 在 y 前
            // 在 u 中：x 在 v 前
            Map<Integer, Integer> xMap = pre.get(x);
            Map<Integer, Integer> uMap = pre.get(u);
            if (xMap.get(u) < xMap.get(y) && uMap.get(x) < uMap.get(v)) {
                cnt++;
                break; // 不开心的原因一个即可
            }
        }
    }
    return cnt;
}
```

#### 复杂度分析

- 时间复杂度：`O(n^2)`
- 空间复杂度：`O(n^2)`

## T3. 连接所有点的最小费用

#### 题目

<div class="notranslate"><p>给你一个<code>points</code>&nbsp;数组，表示 2D 平面上的一些点，其中&nbsp;<code>points[i] = [x<sub>i</sub>, y<sub>i</sub>]</code>&nbsp;。</p>

<p>连接点&nbsp;<code>[x<sub>i</sub>, y<sub>i</sub>]</code> 和点&nbsp;<code>[x<sub>j</sub>, y<sub>j</sub>]</code>&nbsp;的费用为它们之间的 <strong>曼哈顿距离</strong>&nbsp;：<code>|x<sub>i</sub> - x<sub>j</sub>| + |y<sub>i</sub> - y<sub>j</sub>|</code>&nbsp;，其中&nbsp;<code>|val|</code>&nbsp;表示&nbsp;<code>val</code>&nbsp;的绝对值。</p>

<p>请你返回将所有点连接的最小总费用。只有任意两点之间 <strong>有且仅有</strong>&nbsp;一条简单路径时，才认为所有点都已连接。</p>

<p><strong>示例 1：</strong></p>

<p><img style="height:268px; width:214px" src="/img/in-post/leetcode-contest-weekly-206/t3-eg1-1.png" alt=""></p>

<pre><strong>输入：</strong>points = [[0,0],[2,2],[3,10],[5,2],[7,0]]
<strong>输出：</strong>20
<strong>解释：
</strong><img style="height:268px; width:214px" src="/img/in-post/leetcode-contest-weekly-206/t3-eg1-2.png" alt="">
我们可以按照上图所示连接所有点得到最小总费用，总费用为 20 。
注意到任意两个点之间只有唯一一条路径互相到达。
</pre>

<p><strong>示例 2：</strong></p>

<pre><strong>输入：</strong>points = [[3,12],[-2,5],[-4,1]]
<strong>输出：</strong>18
</pre>

<p><strong>示例 3：</strong></p>

<pre><strong>输入：</strong>points = [[0,0],[1,1],[1,0],[-1,1]]
<strong>输出：</strong>4
</pre>

<p><strong>示例 4：</strong></p>

<pre><strong>输入：</strong>points = [[-1000000,-1000000],[1000000,1000000]]
<strong>输出：</strong>4000000
</pre>

<p><strong>示例 5：</strong></p>

<pre><strong>输入：</strong>points = [[0,0]]
<strong>输出：</strong>0
</pre>

<p><strong>提示：</strong></p>

<ul>
	<li><code>1 &lt;= points.length &lt;= 1000</code></li>
	<li><code>-10<sup>6</sup>&nbsp;&lt;= x<sub>i</sub>, y<sub>i</sub> &lt;= 10<sup>6</sup></code></li>
	<li>所有点&nbsp;<code>(x<sub>i</sub>, y<sub>i</sub>)</code>&nbsp;两两不同。</li>
</ul>
</div>

#### 题解思路

- 连接最近的点，优先级队列解决
- 最后可能有好几堆各自为战，并查集解决
- 赛后才知道，这是 [Kruskal 算法](https://zh.wikipedia.org/wiki/%E5%85%8B%E9%B2%81%E6%96%AF%E5%85%8B%E5%B0%94%E6%BC%94%E7%AE%97%E6%B3%95)

#### 参考代码

```java
// 连最近的点
// union set
public int minCostConnectPoints(int[][] points) {
    int n = points.length;
    if (n == 1) return 0;

    // 建模 {i, j, distance}，距离小者先出
    Queue<int[]> queue = new PriorityQueue<>(Comparator.comparingInt(a -> a[2]));
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            if (i == j) continue;
            int dis = dis(points, i, j);
            queue.add(new int[]{i, j, dis});
        }
    }

    int sum = 0;
    UnionFind uf = new UnionFind(n);
    
    // 最后只有一个大集体
    while (uf.count > 1) {
        int[] ele = queue.remove();
        int i = ele[0], j = ele[1], dis = ele[2];
        if (uf.union(i, j)) sum += dis; // 若未连，才能连接
    }
    return sum;
}

private int dis(int[][] arr, int i, int j) {
    int[] a = arr[i], b = arr[j];
    return Math.abs(a[0] - b[0]) + Math.abs(a[1] - b[1]);
}
```

```java
class UnionFind {
    int count = 0;
    private int[] parent;

    public UnionFind(int n) {
        count = n;
        parent = new int[n];
        for (int i = 0; i < n; i++) {
            parent[i] = i;
        }
    }

    public boolean union(int p, int q) {
        int rootP = find(p);
        int rootQ = find(q);
        if (rootP == rootQ) return false;

        parent[rootP] = rootQ;
        count--;
        return true;
    }

    public int find(int p) {
        int root = p;
        while (root != parent[root])
            root = parent[root];
        while (p != parent[p]) {
            int x = p;
            p = parent[p];
            parent[x] = root;
        }
        return root;
    }
}
```

#### 复杂度分析

- 时间复杂度：`O(n^2)`
- 空间复杂度：`O(n^2)`

## T4. 检查字符串是否可以通过排序子字符串得到另一个字符串

#### 题目

<div class="notranslate"><p>给你两个字符串&nbsp;<code>s</code> 和&nbsp;<code>t</code>&nbsp;，请你通过若干次以下操作将字符串&nbsp;<code>s</code>&nbsp;转化成字符串&nbsp;<code>t</code>&nbsp;：</p>

<ul>
	<li>选择 <code>s</code>&nbsp;中一个 <strong>非空</strong>&nbsp;子字符串并将它包含的字符就地 <strong>升序</strong>&nbsp;排序。</li>
</ul>

<p>比方说，对下划线所示的子字符串进行操作可以由&nbsp;<code>"1<strong>4234</strong>"</code>&nbsp;得到&nbsp;<code>"1<strong>2344</strong>"</code>&nbsp;。</p>

<p>如果可以将字符串 <code>s</code>&nbsp;变成 <code>t</code>&nbsp;，返回 <code>true</code>&nbsp;。否则，返回 <code>false</code>&nbsp;。</p>

<p>一个 <strong>子字符串</strong>&nbsp;定义为一个字符串中连续的若干字符。</p>

<p><strong>示例 1：</strong></p>

<pre><strong>输入：</strong>s = "84532", t = "34852"
<strong>输出：</strong>true
<strong>解释：</strong>你可以按以下操作将 s 转变为 t ：
"84<strong>53</strong>2" （从下标 2 到下标 3）-&gt; "84<strong>35</strong>2"
"<strong>843</strong>52" （从下标 0 到下标 2） -&gt; "<strong>348</strong>52"
</pre>

<p><strong>示例 2：</strong></p>

<pre><strong>输入：</strong>s = "34521", t = "23415"
<strong>输出：</strong>true
<strong>解释：</strong>你可以按以下操作将 s 转变为 t ：
"<strong>3452</strong>1" -&gt; "<strong>2345</strong>1"
"234<strong>51</strong>" -&gt; "234<strong>15</strong>"
</pre>

<p><strong>示例 3：</strong></p>

<pre><strong>输入：</strong>s = "12345", t = "12435"
<strong>输出：</strong>false
</pre>

<p><strong>示例 4：</strong></p>

<pre><strong>输入：</strong>s = "1", t = "2"
<strong>输出：</strong>false
</pre>

<p><strong>提示：</strong></p>

<ul>
	<li><code>s.length == t.length</code></li>
	<li><code>1 &lt;= s.length &lt;= 10<sup>5</sup></code></li>
	<li><code>s</code> 和&nbsp;<code>t</code>&nbsp;都只包含数字字符，即&nbsp;<code>'0'</code>&nbsp;到&nbsp;<code>'9'</code> 。</li>
</ul>
</div>

#### 题解思路

- 据说测试用例不足
- 学习自 [坑神的代码](https://www.bilibili.com/video/BV1cA411J74W?p=4)
- **「两两交换」的「向后冒泡」**
- 若某数字后面有大于它的，则不可向后继续冒泡

#### 参考代码

```java
public boolean isTransformable(String s, String t) {
    char[] sarr = s.toCharArray();
    char[] tarr = t.toCharArray();
    int n = sarr.length;

    Stack<Integer>[] pos = new Stack[10]; // 记录每个数字的位置
    for (int v = 0; v <= 9; v++)
        pos[v] = new Stack<>();
    for (int i = 0; i < n; i++)
        pos[sarr[i] - '0'].push(i); // 大的在栈顶

    // 从后向前
    for (int i = n - 1; i >= 0; i--) {
        int d = tarr[i] - '0';
        if (pos[d].isEmpty()) return false;

        // 当前数字 d 之后，不应该有比它大的数
        // 否则会阻拦「向后冒泡」
        for (int j = d + 1; j < 10; j++)
            if (!pos[j].isEmpty() && pos[j].peek() > pos[d].peek())
                return false;
        pos[d].pop(); // 能够换到 t[i] 位置
    }
    return true;
}
```

#### 复杂度分析

- 时间复杂度：`O(n)`
- 空间复杂度：`O(n)`

## 赛后复盘

- T1 希望减少一些时间复杂度，差点还罚时了，卡了好久。其实深度优先或三层循环就完事了
- T2 无脑模拟，比赛时总担心有陷阱，题意也是反复读了好几遍。属于模拟的解法，再配合 `Map` 降到 `O(1)` 查找，仅此而已
- T3 一开始是写错方向了，WA 了一次才知道：两堆点没连在一起。果断转并查集，稳定输出
- T4 比赛思考应该是逆序对的问题，时间不够了，后续再看题解吧
    - 思路很巧妙，关键点还是在「两两交换」的「向后冒泡」

总体上自己是满意的，思考的过程还比较清晰，代码输出也更稳定了，不会出现一些低级错误。

测试用例的能力还是需要提高，现在甚至都懒得去造，其实这是一个非常好的锻炼更全局思维的方法，也是编程的必备素质。
