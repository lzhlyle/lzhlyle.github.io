---
layout:     post
title:      测试用例不是吃素的
subtitle:   LeetCode 第 205 场周赛
date:       2020-09-06
author:     lyle
header-img: "img/in-post/leetcode-contest-weekly-205/banner.jpg"
tags:
    - LeetCode 竞赛
    - 二分
    - 并查集
    - 图
    - 贪心
---

> 13 分，1000+ 名

## 题目

1. [替换所有的问号](#t1-替换所有的问号)
2. [数的平方等于两数乘积的方法数](#t2-数的平方等于两数乘积的方法数)
3. [避免重复字母的最小删除成本](#t3-避免重复字母的最小删除成本)
4. [保证图可完全遍历](#t4-保证图可完全遍历)
5. [赛后复盘](#赛后复盘)

## T1. 替换所有的问号

#### 题目

<div class="notranslate"><p>给你一个仅包含小写英文字母和 <code>'?'</code> 字符的字符串 <code>s</code>，请你将所有的 <code>'?'</code> 转换为若干小写字母，使最终的字符串不包含任何 <strong>连续重复</strong> 的字符。</p>

<p>注意：你 <strong>不能</strong> 修改非 <code>'?'</code> 字符。</p>

<p>题目测试用例保证 <strong>除</strong> <code>'?'</code> 字符 <strong>之外</strong>，不存在连续重复的字符。</p>

<p>在完成所有转换（可能无需转换）后返回最终的字符串。如果有多个解决方案，请返回其中任何一个。可以证明，在给定的约束条件下，答案总是存在的。</p>

<p>&nbsp;</p>

<p><strong>示例 1：</strong></p>

<pre><strong>输入：</strong>s = "?zs"
<strong>输出：</strong>"azs"
<strong>解释：</strong>该示例共有 25 种解决方案，从 "azs" 到 "yzs" 都是符合题目要求的。只有 "z" 是无效的修改，因为字符串 "zzs" 中有连续重复的两个 'z' 。</pre>

<p><strong>示例 2：</strong></p>

<pre><strong>输入：</strong>s = "ubv?w"
<strong>输出：</strong>"ubvaw"
<strong>解释：</strong>该示例共有 24 种解决方案，只有替换成 "v" 和 "w" 不符合题目要求。因为 "ubvvw" 和 "ubvww" 都包含连续重复的字符。
</pre>

<p><strong>示例 3：</strong></p>

<pre><strong>输入：</strong>s = "j?qg??b"
<strong>输出：</strong>"jaqgacb"
</pre>

<p><strong>示例 4：</strong></p>

<pre><strong>输入：</strong>s = "??yw?ipkj?"
<strong>输出：</strong>"acywaipkja"
</pre>

<p>&nbsp;</p>

<p><strong>提示：</strong></p>

<ul>
	<li>
	<p><code>1 &lt;= s.length&nbsp;&lt;= 100</code></p>
	</li>
	<li>
	<p><code>s</code> 仅包含小写英文字母和 <code>'?'</code> 字符</p>
	</li>
</ul>
</div>

#### 题解思路

- 最多需要三个字母替换：`a`，`b`，`c`
    - 参考代码中 `get(char)` 使用字典序替换处理
- 特殊处理第一个、最后一个
- 中间的都需要看左右相邻

#### 参考代码

```java
public String modifyString(String s) {
    if (s.equals("?")) return "a";

    char[] arr = s.toCharArray();
    for (int i = 0; i < arr.length; i++) {
        if (arr[i] == '?') {
            if (i == 0) arr[i] = get(arr[i + 1]);
            else if (i == arr.length - 1) arr[i] = get(arr[i - 1]);
            else arr[i] = get(arr[i - 1], arr[i + 1]);
        }
    }
    return new String(arr);
}

private char get(char l, char r) {
    if (r == '?') return get(l);
    char res = get(r);
    while (res == r || res == l)
        res = get(res);
    return res;
}

private char get(char nb) {
    if (nb == '?') return 'a';
    if (nb == 'z') return 'a';
    return (char) (nb + 1);
}
```

#### 复杂度分析

- 时间复杂度：`O(n)`
- 空间复杂度：`O(1)`

## T2. 数的平方等于两数乘积的方法数

#### 题目

<div class="notranslate"><p>给你两个整数数组 <code>nums1</code> 和 <code>nums2</code> ，请你返回根据以下规则形成的三元组的数目（类型 1 和类型 2 ）：</p>

<ul>
	<li>类型 1：三元组 <code>(i, j, k)</code> ，如果 <code>nums1[i]<sup>2</sup>&nbsp;== nums2[j] * nums2[k]</code> 其中 <code>0 &lt;= i &lt; nums1.length</code> 且 <code>0 &lt;= j &lt; k &lt; nums2.length</code></li>
	<li>类型 2：三元组 <code>(i, j, k)</code> ，如果 <code>nums2[i]<sup>2</sup>&nbsp;== nums1[j] * nums1[k]</code> 其中 <code>0 &lt;= i &lt; nums2.length</code> 且 <code>0 &lt;= j &lt; k &lt; nums1.length</code></li>
</ul>

<p>&nbsp;</p>

<p><strong>示例 1：</strong></p>

<pre><strong>输入：</strong>nums1 = [7,4], nums2 = [5,2,8,9]
<strong>输出：</strong>1
<strong>解释：</strong>类型 1：(1,1,2), nums1[1]^2 = nums2[1] * nums2[2] (4^2 = 2 * 8)</pre>

<p><strong>示例 2：</strong></p>

<pre><strong>输入：</strong>nums1 = [1,1], nums2 = [1,1,1]
<strong>输出：</strong>9
<strong>解释：</strong>所有三元组都符合题目要求，因为 1^2 = 1 * 1
类型 1：(0,0,1), (0,0,2), (0,1,2), (1,0,1), (1,0,2), (1,1,2), nums1[i]^2 = nums2[j] * nums2[k]
类型 2：(0,0,1), (1,0,1), (2,0,1), nums2[i]^2 = nums1[j] * nums1[k]
</pre>

<p><strong>示例 3：</strong></p>

<pre><strong>输入：</strong>nums1 = [7,7,8,3], nums2 = [1,2,9,7]
<strong>输出：</strong>2
<strong>解释：</strong>有两个符合题目要求的三元组
类型 1：(3,0,2), nums1[3]^2 = nums2[0] * nums2[2]
类型 2：(3,0,1), nums2[3]^2 = nums1[0] * nums1[1]
</pre>

<p><strong>示例 4：</strong></p>

<pre><strong>输入：</strong>nums1 = [4,7,9,11,23], nums2 = [3,5,1024,12,18]
<strong>输出：</strong>0
<strong>解释：</strong>不存在符合题目要求的三元组
</pre>

<p>&nbsp;</p>

<p><strong>提示：</strong></p>

<ul>
	<li><code>1 &lt;= nums1.length, nums2.length &lt;= 1000</code></li>
	<li><code>1 &lt;= nums1[i], nums2[i] &lt;= 10^5</code></li>
</ul>
</div>

#### 题解思路

- 枚举 `i`、`j`，利用二分查找 `j + 1` 第一个满足的 `k`
- 排序后可用二分快速查找
    - 只需求总方案数，故重排序不影响结果
- `Map` 记录排序后的 `(value, indexList)`，用于快速计算 `k` 之后有几个相同值

#### 参考代码

```java
public int numTriplets(int[] nums1, int[] nums2) {
    Arrays.sort(nums1);
    Arrays.sort(nums2);

    Map<Integer, List<Integer>> map1 = getMap(nums1), map2 = getMap(nums2);

    return cnt(nums1, nums2, map1, map2) + cnt(nums2, nums1, map2, map1);
}

private int cnt(int[] nums1, int[] nums2, Map<Integer, List<Integer>> map1, Map<Integer, List<Integer>> map2) {
    int cnt = 0;
    int last = 0;
    
    // 枚举 i
    for (int i = 0; i < nums1.length; i++) {
        int v = nums1[i];
        if (i > 0 && v == nums1[i - 1]) { // nums1 同值，快速计算
            cnt += last;
            continue;
        }

        long v2 = (long) v * (long) v;
        int curr = 0; // 当前 i 的结果
        
        // 枚举 j
        for (int j = 0; j < nums2.length - 1; j++) {
        
            // 二分查找 k：从 j + 1 开始，第一个满足的
            int l = j + 1, r = nums2.length - 1;
            while (l < r) {
                int mid = l + (r - l) / 2;
                if ((long) nums2[mid] * (long) nums2[j] < v2) l = mid + 1;
                else r = mid;
            }
            if ((long) nums2[l] * (long) nums2[j] == v2) {
                // 还要看 l 后面有几个与 nums2[l] 相同
                List<Integer> list = map2.get(nums2[l]);
                curr += list.get(list.size() - 1) - l + 1;
            }
        }
        
        cnt += curr; // 累计
        last = curr; // 记录缓存
    }
    // System.out.println();
    return cnt;
}

// (value, indexList)
private Map<Integer, List<Integer>> getMap(int[] arr) {
    Map<Integer, List<Integer>> map = new HashMap<>();
    for (int i = 0; i < arr.length; i++) {
        if (!map.containsKey(arr[i]))
            map.put(arr[i], new ArrayList<>());
        map.get(arr[i]).add(i);
    }
    return map;
}
```

#### 复杂度分析

- 时间复杂度：`O(mnlog(n) + nmlog(m))`
- 空间复杂度：`O(m + n)`

## T3. 避免重复字母的最小删除成本

#### 题目

<div class="notranslate"><p>给你一个字符串 <code>s</code> 和一个整数数组 <code>cost</code> ，其中 <code>cost[i]</code> 是从 <code>s</code> 中删除字符 <code>i</code> 的代价。</p>

<p>返回使字符串任意相邻两个字母不相同的最小删除成本。</p>

<p>请注意，删除一个字符后，删除其他字符的成本不会改变。</p>

<p>&nbsp;</p>

<p><strong>示例 1：</strong></p>

<pre><strong>输入：</strong>s = "abaac", cost = [1,2,3,4,5]
<strong>输出：</strong>3
<strong>解释：</strong>删除字母 "a" 的成本为 3，然后得到 "abac"（字符串中相邻两个字母不相同）。
</pre>

<p><strong>示例 2：</strong></p>

<pre><strong>输入：</strong>s = "abc", cost = [1,2,3]
<strong>输出：</strong>0
<strong>解释：</strong>无需删除任何字母，因为字符串中不存在相邻两个字母相同的情况。
</pre>

<p><strong>示例 3：</strong></p>

<pre><strong>输入：</strong>s = "aabaa", cost = [1,2,3,4,1]
<strong>输出：</strong>2
<strong>解释：</strong>删除第一个和最后一个字母，得到字符串 ("aba") 。
</pre>

<p>&nbsp;</p>

<p><strong>提示：</strong></p>

<ul>
	<li><code>s.length == cost.length</code></li>
	<li><code>1 &lt;= s.length, cost.length &lt;= 10^5</code></li>
	<li><code>1 &lt;= cost[i] &lt;=&nbsp;10^4</code></li>
	<li><code>s</code> 中只含有小写英文字母</li>
</ul>
</div>

#### 题解思路

- 两两比较，删成本大的
- `last` 删后上一个字母，删谁都一样
- `lastIndex` 删后上一个字母的位置，删谁有不同

#### 参考代码

```java
public int minCost(String s, int[] cost) {
    char[] arr = s.toCharArray();
    int n = arr.length;
    if (n == 1) return 0;

    int res = 0;
    char last = '0';
    int lastIndex = -1;
    for (int i = 0; i < n; i++) {
        if (arr[i] == last) {
            if (cost[i] < cost[lastIndex]) {
                // 删 arr[i]
                res += cost[i];
                // lastIndex = lastIndex 无变化
            } else {
                // 删 arr[lastIndex]
                res += cost[lastIndex];
                lastIndex = i; // 变化
            }
        } else lastIndex = i; // 更新
        last = arr[i]; // 总是更新
    }
    return res;
}
```

#### 复杂度分析

- 时间复杂度：`O(n)`
- 空间复杂度：`O(1)`

## T4. 保证图可完全遍历

#### 题目

<div class="notranslate"><p>Alice 和 Bob 共有一个无向图，其中包含 n 个节点和 3&nbsp; 种类型的边：</p>

<ul>
	<li>类型 1：只能由 Alice 遍历。</li>
	<li>类型 2：只能由 Bob 遍历。</li>
	<li>类型 3：Alice 和 Bob 都可以遍历。</li>
</ul>

<p>给你一个数组 <code>edges</code> ，其中 <code>edges[i] = [type<sub>i</sub>, u<sub>i</sub>, v<sub>i</sub>]</code>&nbsp;表示节点 <code>u<sub>i</sub></code> 和 <code>v<sub>i</sub></code> 之间存在类型为 <code>type<sub>i</sub></code> 的双向边。请你在保证图仍能够被 Alice和 Bob 完全遍历的前提下，找出可以删除的最大边数。如果从任何节点开始，Alice 和 Bob 都可以到达所有其他节点，则认为图是可以完全遍历的。</p>

<p>返回可以删除的最大边数，如果 Alice 和 Bob 无法完全遍历图，则返回 -1 。</p>

<p>&nbsp;</p>

<p><strong>示例 1：</strong></p>

<p><strong><img style="height: 191px; width: 179px;" src="https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/09/06/5510ex1.png" alt=""></strong></p>

<pre><strong>输入：</strong>n = 4, edges = [[3,1,2],[3,2,3],[1,1,3],[1,2,4],[1,1,2],[2,3,4]]
<strong>输出：</strong>2
<strong>解释：</strong>如果删除<strong> </strong>[1,1,2] 和 [1,1,3] 这两条边，Alice 和 Bob 仍然可以完全遍历这个图。再删除任何其他的边都无法保证图可以完全遍历。所以可以删除的最大边数是 2 。
</pre>

<p><strong>示例 2：</strong></p>

<p><strong><img style="height: 190px; width: 178px;" src="https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/09/06/5510ex2.png" alt=""></strong></p>

<pre><strong>输入：</strong>n = 4, edges = [[3,1,2],[3,2,3],[1,1,4],[2,1,4]]
<strong>输出：</strong>0
<strong>解释：</strong>注意，删除任何一条边都会使 Alice 和 Bob 无法完全遍历这个图。
</pre>

<p><strong>示例 3：</strong></p>

<p><strong><img style="height: 190px; width: 178px;" src="https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/09/06/5510ex3.png" alt=""></strong></p>

<pre><strong>输入：</strong>n = 4, edges = [[3,2,3],[1,1,2],[2,3,4]]
<strong>输出：</strong>-1
<strong>解释：</strong>在当前图中，Alice 无法从其他节点到达节点 4 。类似地，Bob 也不能达到节点 1 。因此，图无法完全遍历。</pre>

<p>&nbsp;</p>

<p><strong>提示：</strong></p>

<ul>
	<li><code>1 &lt;= n &lt;= 10^5</code></li>
	<li><code>1 &lt;= edges.length &lt;= min(10^5, 3 * n * (n-1) / 2)</code></li>
	<li><code>edges[i].length == 3</code></li>
	<li><code>1 &lt;= edges[i][0] &lt;= 3</code></li>
	<li><code>1 &lt;= edges[i][1] &lt; edges[i][2] &lt;= n</code></li>
	<li>所有元组 <code>(type<sub>i</sub>, u<sub>i</sub>, v<sub>i</sub>)</code> 互不相同</li>
</ul>
</div>

#### 题解思路

- 可用「逐渐添加边」的思路做
    - 遍历边，若两点已能连通，则无需此边
- 并查集：判断两点是否已经连通
- 图：利用入度快速判断是否可全通
- 贪心：优先使用共享路径

#### 参考代码

```java
class UnionFind {
    private int count;
    private int[] parent;

    public UnionFind(int n) {
        count = n;
        parent = new int[n + 1];
        for (int i = 1; i <= n; i++)
            parent[i] = i;
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

```java
public int maxNumEdgesToRemove(int n, int[][] edges) {
    if (edges.length == 1) return 0;

    // 各点的入度情况
    int[][] ins = new int[n + 1][3 + 1];
    for (int[] e : edges) {
        int type = e[0], a = e[1], b = e[2];
        ins[a][type]++;
        ins[b][type]++;
    }

    // 检查无法完全遍历
    for (int i = 1; i <= n; i++) {
        int[] in = ins[i];
        if (in[3] == 0 && (in[1] == 0 || in[2] == 0))
            return -1;
    }

    // 可完全遍历
    int res = 0;
    // 并查集
    UnionFind ufa = new UnionFind(n), ufb = new UnionFind(n);
    // 贪心：先选共用 e[0] desc
    Arrays.sort(edges, (e1, e2) -> e2[0] - e1[0]);
    for (int[] e : edges) {
        int type = e[0], a = e[1], b = e[2];
        if (type == 1) {
            if (!ufa.union(a, b)) res++;
        } else if (type == 2) {
            if (!ufb.union(a, b)) res++;
        } else {
            boolean ua = ufa.union(a, b), ub = ufb.union(a, b);
            if (!ua && !ub) res++; // Alice, Bob 都不再需要这条边
        }
    }
    return res;
}
```

#### 复杂度分析

- 时间复杂度：`O(n)`
- 空间复杂度：`O(n)`

## 赛后复盘

- T1 
- T2 
- T3 
- T4 


