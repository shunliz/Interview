### 1. 什么是 LCSs？

　　什么是 LCSs? 好多博友看到这几个字母可能比较困惑，因为这是我自己对两个常见问题的统称，它们分别为最长公共子序列问题（Longest-Common-Subsequence）和最长公共子串（Longest-Common-Substring）问题。这两个问题非常的相似，所以对不熟悉的同学来说，有时候很容易被混淆。下面让我们去好好地理解一下两者的区别吧。

#### 1.1 子序列 vs 子串

**子序列是有序的，但不一定是连续，作用对象是序列。**

　　例如：序列 X = &lt;B, C, D, B&gt; 是序列 Y = &lt;A, B, C, B, D, A, B&gt; 的子序列，对应的下标序列为 &lt;2, 3, 5, 7&gt;。

**子串是有序且连续的，左右对象是字符串。**

　　例如 a = abcd 是 c = aaabcdddd 的一个子串；但是 b = acdddd 就不是 c 的子串。

#### 1.2 最长公共子序列 vs 最长公共子串

　　最长公共子序列和最长公共子串是常见的两种问题，虽然两者问题很相似，也均可以根据动态规划进行求解，但是两者的本质是不同的。

　　最长公共子序列问题是针对给出的两个序列，求两个序列最长的公共子序列。

　　最长公共子串问题是针对给出的两个字符串，求两个字符串最长的公共子串（有关字符串匹配相关算法可以转至博客《[\[Algorithm\] 字符串匹配算法——KMP算法](http://www.cnblogs.com/maybe2030/p/4633153.html)》）。



### 2. 动态规划方法求解LCSs

　　前面提到，动态规划方法均可以用到最长公共子序列和最长公共子串问题当中，在这里我们就不一一进行求解了。我们以最长公共子序列为例，介绍一下如何利用动态规划的思想来解决 LCSs。  


　　给定两个序列，找出在两个序列中同时出现的最长子序列的**长度**。对于每一个序列而言，其均具有 ![](http://mathjax.cnblogs.com/2_6_1/fonts/HTML-CSS/TeX/png/Math/Italic/400/0061.png?rev=2.6.1)![](http://mathjax.cnblogs.com/2_6_1/fonts/HTML-CSS/TeX/png/Math/Italic/283/006D.png?rev=2.6.1)中子序列，因此采用暴力算法的时间复杂度是指数级的，这显然不是一种好的解决方案。

　　下面我们看一下，如何使用动态规划的思想来解决最大公共子序列问题。

　　首先考虑最大公共子序列问题是否满足动态规划问题的两个基本特性：

**1. 最优子结构：**

设输入序列是X \[0 .. m-1\] 和 Y \[0 .. n-1\]，长度分别为 m 和 n。和设序列 L\(X \[0 .. m-1\]，Y\[0 .. n-1\]\) 是这两个序列的 LCS 的长度，以下为 L\(X \[0 .. M-1\]，Y \[0 .. N-1\]\) 的递归定义：

　　1）如果两个序列的最后一个元素匹配（即X \[M-1\] == Y \[N-1\]）

　　则：L（X \[0 .. M-1\]，Y \[0 .. N-1\]）= 1 + L（X \[0 .. M-2\]，Y \[0 .. N-1\]）

　　2）如果两个序列的最后字符不匹配（即X \[M-1\] != Y \[N-1\]）  
　　则：L\(X \[0 .. M-1\]，Y \[0 .. N-1\]\) = MAX\(L\(X \[0 .. M-2\]，Y \[0 .. N-1\]\)，L\(X \[0 .. M-1\]，Y \[0 .. N-2\]\)\)

　　通过如下具体实例来更好地理解一下：

1）考虑输入子序列 &lt;AGGTAB&gt; 和 &lt;GXTXAYB&gt;。最后一个字符匹配的字符串。这样的 LCS 的长度可以写成：

L\(&lt;AGGTAB&gt;, &lt;GXTXAYB&gt;\) = 1 + L\(&lt;AGGTA&gt;, &lt;GXTXAY&gt;\)

　　2）考虑输入字符串“ABCDGH”和“AEDFHR。最后字符不为字符串相匹配。这样的LCS的长度可以写成：

L\(&lt;ABCDGH&gt;, &lt;AEDFHR&gt;\) = MAX \( L\(&lt;ABCDG&gt;, &lt;AEDFHR&gt;\), L\(&lt;ABCDGH&gt;, &lt;AEDFH&gt;\) \)

　　因此，LCS问题有最优子结构性质。

**2. 重叠子问题：**

　　很明显，基于上述的分析，LCS 很多子问题也都共享子子问题，因此可以对其进行递归求解。具体的算法时间度为 O\(m\*n\)，可以优化至 O\(m+n\)。

　　下图给出了回溯法找出LCS的过程：

![](http://images2015.cnblogs.com/blog/764050/201605/764050-20160508144903780-173154494.gif)

　　具体的C++实现代码如下：

```cpp
/ *动态规划实现的LCS问题* /
#include<stdio.h>
#include<stdlib.h>

int max(int a, int b);

/* Returns length of LCS for X[0..m-1], Y[0..n-1] */
int lcs( char *X, char *Y, int m, int n )
{
   int L[m+1][n+1];
   int i, j;

   /* Following steps build L[m+1][n+1] in bottom up fashion. Note 
      that L[i][j] contains length of LCS of X[0..i-1] and Y[0..j-1] */
   for (i=0; i<=m; i++)
   {
     for (j=0; j<=n; j++)
     {
       if (i == 0 || j == 0)
         L[i][j] = 0;

       else if (X[i-1] == Y[j-1])
         L[i][j] = L[i-1][j-1] + 1;

       else
         L[i][j] = max(L[i-1][j], L[i][j-1]);
     }
   }

   /* L[m][n] contains length of LCS for X[0..n-1] and Y[0..m-1] */
   return L[m][n];
}

/* Utility function to get max of 2 integers */
int max(int a, int b)
{
    return (a > b)? a : b;
}

/*测试上面的函数 */
int main()
{
  char X[] = "AGGTAB";
  char Y[] = "GXTXAYB";

  int m = strlen(X);
  int n = strlen(Y);

  printf("Length of LCS is %d\n", lcs( X, Y, m, n ) );

  getchar();
  return 0;
}
```



