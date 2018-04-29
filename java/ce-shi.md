* [前言](#前言)
* [1. 小米-小米Git](#1-小米-小米git)
* [2. 小米-懂二进制](#2-小米-懂二进制)
* [3. 小米-中国牛市](#3-小米-中国牛市)
* [4. 微软-LUCKY STRING](#4-微软-lucky-string)
* [5. 微软-Numeric Keypad](#5-微软-numeric-keypad)
* [6. 微软-Spring Outing](#6-微软-spring-outing)
* [7. 微软-S-expression](#7-微软-s-expression)
* [8. 华为-最高分是多少](#8-华为-最高分是多少)
* [9. 华为-简单错误记录](#9-华为-简单错误记录)
* [10. 华为-扑克牌大小](#10-华为-扑克牌大小)
* [11. 去哪儿-二分查找](#11-去哪儿-二分查找)
* [12. 去哪儿-首个重复字符](#12-去哪儿-首个重复字符)
* [13. 去哪儿-寻找Coder](#13-去哪儿-寻找coder)
* [14. 美团-最大差值](#14-美团-最大差值)
* [15. 美团-棋子翻转](#15-美团-棋子翻转)
* [16. 美团-拜访](#16-美团-拜访)
* [17. 美团-直方图内最大矩形](#17-美团-直方图内最大矩形)
* [18. 美团-字符串计数](#18-美团-字符串计数)
* [19. 美团-平均年龄](#19-美团-平均年龄)
* [20. 百度-罪犯转移](#20-百度-罪犯转移)
* [22. 百度-裁减网格纸](#22-百度-裁减网格纸)
* [23. 百度-钓鱼比赛](#23-百度-钓鱼比赛)
* [24. 百度-蘑菇阵](#24-百度-蘑菇阵)

# 前言

省略的代码：

```java
import java.util.*;
```

```java
public class Solution {
}
```

```java
public class Main {
public static void main(String[] args) {
Scanner in = new Scanner(System.in);
while (in.hasNext()) {
}
}
}
```

# 1. 小米-小米Git

- 重建多叉树
- 使用 LCA

```java
private class TreeNode {
int id;
List<TreeNode> childs = new ArrayList<>();

TreeNode(int id) {
this.id = id;
}
}

public int getSplitNode(String[] matrix, int indexA, int indexB) {
int n = matrix.length;
boolean[][] linked = new boolean[n][n]; // 重建邻接矩阵
for (int i = 0; i < n; i++) {
for (int j = 0; j < n; j++) {
linked[i][j] = matrix[i].charAt(j) == '1';
}
}
TreeNode tree = constructTree(linked, 0);
TreeNode ancestor = LCA(tree, new TreeNode(indexA), new TreeNode(indexB));
return ancestor.id;
}

private TreeNode constructTree(boolean[][] linked, int root) {
TreeNode tree = new TreeNode(root);
for (int i = 0; i < linked[root].length; i++) {
if (linked[root][i]) {
linked[i][root] = false; // 因为题目给的邻接矩阵是双向的，在这里需要把它转为单向的
tree.childs.add(constructTree(links, i));
}
}
return tree;
}

private TreeNode LCA(TreeNode root, TreeNode p, TreeNode q) {
if (root == null || root.id == p.id || root.id == q.id) return root;
TreeNode ancestor = null;
int cnt = 0;
for (int i = 0; i < root.childs.size(); i++) {
TreeNode tmp = LCA(root.childs.get(i), p, q);
if (tmp != null) {
ancestor = tmp;
cnt++;
}
}
return cnt == 2 ? root : ancestor;
}
```