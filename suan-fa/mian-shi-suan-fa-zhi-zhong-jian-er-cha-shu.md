1

**题目**

已知前序遍历为{1,2,4,7,3,5,6,8}，中序遍历为{4,7,2,1,5,3,8,6}，它的二叉树是怎么样的？

  


2

**基础巩固**

根据上述题目所述，我们已知前序遍历和中序遍历，回顾一下，什么是前序遍历？什么是中序遍历？

  


**2.1 前序遍历**

  


前序遍历一颗二叉树，首先输出根节点，然后输出左子节点，最后输出右子节点。

  


比如，遍历一下二叉树：

  


![](https://mmbiz.qpic.cn/mmbiz_gif/ouvf8kz8iaAtPC8qX88rFhG95eRUmqmFSiazEC9xHGImABuG4DzllWnkU2Ts0AEeudPS7FoM1MhZy05yq0SySbxA/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

颜色变深表示遍历，突出表示输出

  


**2.2 中序遍历**  


  


中序遍历一棵二叉树，首先输出左子节点，然后输出输出根节点，最后右子节点。

  


以上边二叉树为例，通过中序遍历输出。

  


![](https://mmbiz.qpic.cn/mmbiz_gif/ouvf8kz8iaAtPC8qX88rFhG95eRUmqmFSPLCAUZ3qiaqsickMVKYmcAXCbu3ibcwmxoP1vSFSesnKWuiaChoqXLZ1iaQ/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

  


3

**解题思路**

既然我们知道了二叉树如何进行前序遍历和中序遍历了，那么已知前序遍历和中序遍历如何反推二叉树呢？

  


那么问题来了，只知道前序遍历能不能反推二叉树呢？我们就试一下，比如题目中所述，`{1,2,4,7,3,5,6,8}`，根据前序遍历，根、左、右，1 肯定是 根节点，那么一下`2,4,7.....`哪些是左子节点呢？左子节点有几个呢？很显然我们是不知道的，由此可以得出，只知道前序遍历是不可能反推出二叉树的，中序遍历也是如此，自己可以尝试一下。

  


那么前序遍历和中序遍历可不可以？那我们要试一下，我们上边通过前序遍历找到第一个根节点就是 1，如图

![](https://mmbiz.qpic.cn/mmbiz_png/ouvf8kz8iaAtPC8qX88rFhG95eRUmqmFS23pbG8JIpqpRQmdC8RYYBfECLibL5AlYoicvrXkejU2w2txwdkQjnmvQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

中序遍历`{4,7,2,1,5,3,8,6}`的规律又是左、根、右，所以 1 结点在中序遍历中为根，它的左边就是所有左子节点`4,7,2`，右边为所有的右子节点`5,3,8,6`。

![](https://mmbiz.qpic.cn/mmbiz_png/ouvf8kz8iaAtPC8qX88rFhG95eRUmqmFSKFmOFvhQnRuqBXB0apngLNNtrlY3p6POUTX6TxnPrjgMT6aRiazHhtg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

此时我们已经划分左右子节点了，但是左边的子节点中哪些又是根节点呢？我们再回到前序遍历中，根据前序遍历的特点，根、左、右，在从子节点进行划分，那么 1 结点中的子节点谁为根节点呢？

  


我们一眼就能看出来，就是 2 结点，那么剩余的 4,7 左右结点怎么分呢？我们根据上述再回到中序遍历，找到 2 根节点，根据中序遍历左、根、右的特点，找到 2 结点，那左边的就是左子节点，右边的就是右子节点，我们可以看到，左边有两个数，正是 4 和 7 结点。

  


右边只有 1 结点，1 结点我们刚刚说了，是根节点，所以以 2 为根节点是没有右子节点的，剩下的数字也是同样的方式区分，自己可以试一下，动手画一画。

![](https://mmbiz.qpic.cn/mmbiz_png/ouvf8kz8iaAtPC8qX88rFhG95eRUmqmFSVVnFvbZpATyS6FOUnlL0ZMRfN59yyH5qSWLGpLbpkV2nKLX8yqcQibA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们仔细发现，其实整个层层往下，以及左右，都是相同的解决方式，而且大问题可以分解为小问题，我们下意识就应该想起小鹿之前分享过的知识，那就是递归，既然是递归，就应该有终止条件，终止条件就是当子节点为空时，此时递归结束。

  


4

**测试用例**

  


我们之前的文章强调过，手写代码之前，一定先把测试用例想好，为了能够在手写代码的时候考虑到边界情况，还为了防止你到时候面试涂涂改改。

  


**4.1 普通测试**

  


完全二叉树、非完全二叉树。

  


**4.2 特殊测试**

  


只有左子节点二叉树，只有右子节点、只有一个结点的二叉树 —— 特殊二叉树测试。

  


**4.3 输入测试**

  


空树、空指针`null`、前序和中序不匹配。

  


5

**代码实现**

  


**JavaScript 版本**

![](https://mmbiz.qpic.cn/mmbiz_png/ouvf8kz8iaAtPC8qX88rFhG95eRUmqmFS6IYCymy2icvg5LPZE0d2xlTurtaSWCxr7bn7EVUYGLkbn7MUYspgJQg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


**Java 版本**

![](https://mmbiz.qpic.cn/mmbiz_png/ouvf8kz8iaAtPC8qX88rFhG95eRUmqmFSbDLCS7MdXrhibxbmJO8LP7IzwvYic8zNGNib2qWL11WibxHzn8CbSUr56A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


**C 语言版本**

![](https://mmbiz.qpic.cn/mmbiz_png/ouvf8kz8iaAtPC8qX88rFhG95eRUmqmFSEvRZeWqtAtCKwxp9dOarqw0Npn75dokUuT0Vm8ADpeGowwICNjiaaxQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


**Python 版本**

![](https://mmbiz.qpic.cn/mmbiz_png/ouvf8kz8iaAtPC8qX88rFhG95eRUmqmFSXNekPn7H2g8orGraPRlJr2Df1KI5pfsnia0lY9DIezCibsufStXKu3MQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

