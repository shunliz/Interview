### 1 字符串匹配

　　字符串匹配是计算机的基本任务之一。

　　字符串匹配是什么？举例来说，有一个字符串"BBC ABCDAB ABCDABCDABDE"，我想知道，里面是否包含另一个字符串"ABCDABD"？

　　许多算法可以完成这个任务，Knuth-Morris-Pratt算法（简称KMP）是最常用的之一。它以三个发明者命名，起头的那个K就是著名科学家Donald Knuth（《计算机程序设计艺术》的作者）。

### 2 KMP算法

　　这个算法不太容易理解，网上有很多解释，但读起来都很费劲。直到读到Jake Boxer的文章，我才真正理解这种算法。下面，我用自己的语言，试图写一篇比较好懂的KMP算法解释。

　　1.

![](http://images.cnitblog.com/news/331782/201305/02140954-a6364372f7cb42219c25c9a42ca91f2e.png)

　　首先，字符串"BBC ABCDAB ABCDABCDABDE"的第一个字符与搜索词"ABCDABD"的第一个字符，进行比较。因为B与A不匹配，所以搜索词后移一位。

　　2.

![](http://images.cnitblog.com/news/331782/201305/02140954-4d282a1fa8e644f69c0ad9a96b8d4e0f.png)

　　因为B与A不匹配，搜索词再往后移。

　　3.

![](http://images.cnitblog.com/news/331782/201305/02140954-403557bcb10e41dbbce9434da36d105c.png)

　　就这样，直到字符串有一个字符，与搜索词的第一个字符相同为止。

　　4.

![](http://images.cnitblog.com/news/331782/201305/02140954-c084752484ac48eaa930a765095ed139.png)

　　接着比较字符串和搜索词的下一个字符，还是相同。

　　5.

![](http://images.cnitblog.com/news/331782/201305/02140955-99c6a1d36b9e418c96de092b8ca1f5ed.png)

　　直到字符串有一个字符，与搜索词对应的字符不相同为止。

　　6.

![](http://images.cnitblog.com/news/331782/201305/02140955-90c93e1401c843148af09b472400e613.png)

　　这时，最自然的反应是，将搜索词整个后移一位，再从头逐个比较。这样做虽然可行，但是效率很差，因为你要把"搜索位置"移到已经比较过的位置，重比一遍。

　　7.

![](http://images.cnitblog.com/news/331782/201305/02140955-4da97276b7aa4ab0bc760623d8f59434.png)

　　一个基本事实是，当空格与D不匹配时，你其实知道前面六个字符是"ABCDAB"。KMP算法的想法是，设法利用这个已知信息，不要把"搜索位置"移回已经比较过的位置，继续把它向后移，这样就提高了效率。

　　8.

![](http://images.cnitblog.com/news/331782/201305/02140956-16bec39296b74bcaaf0627e20b846120.png)

　　怎么做到这一点呢？可以针对搜索词，算出一张《部分匹配表》（Partial Match Table）。这张表是如何产生的，后面再介绍，这里只要会用就可以了。

　　9.

![](http://images.cnitblog.com/news/331782/201305/02140955-14131127b6574a3391b337024614306a.png)

　　已知空格与D不匹配时，前面六个字符"ABCDAB"是匹配的。查表可知，最后一个匹配字符B对应的"部分匹配值"为2，因此按照下面的公式算出向后移动的位数：

> 移动位数 = 已匹配的字符数 - 对应的部分匹配值

　　因为 6 - 2 等于4，所以将搜索词向后移动4位。

　　10.

![](http://images.cnitblog.com/news/331782/201305/02140955-9707cee873fd452291d0638b295a3b9d.png)

　　因为空格与Ｃ不匹配，搜索词还要继续往后移。这时，已匹配的字符数为2（"AB"），对应的"部分匹配值"为0。所以，移动位数 = 2 - 0，结果为 2，于是将搜索词向后移2位。

　　11.

![](http://images.cnitblog.com/news/331782/201305/02140959-bf69aab69d114c1da30362d3aeb57bdd.png)

　　因为空格与A不匹配，继续后移一位。

　　12.

![](http://images.cnitblog.com/news/331782/201305/02140956-7b7bf61a842f46a8949cc1d084a56f79.png)

　　逐位比较，直到发现C与D不匹配。于是，移动位数 = 6 - 2，继续将搜索词向后移动4位。

　　13.

![](http://images.cnitblog.com/news/331782/201305/02140957-b6c5a85c046244bf8805b9c0fb88e829.png)

　　逐位比较，直到搜索词的最后一位，发现完全匹配，于是搜索完成。如果还要继续搜索（即找出全部匹配），移动位数 = 7 - 0，再将搜索词向后移动7位，这里就不再重复了。

　　14.

![](http://images.cnitblog.com/news/331782/201305/02140956-8bf1c8f8454a4e269e287a2f7afe1a45.png)

　　下面介绍《部分匹配表》是如何产生的。

　　首先，要了解两个概念："前缀"和"后缀"。 "前缀"指除了最后一个字符以外，一个字符串的全部头部组合；"后缀"指除了第一个字符以外，一个字符串的全部尾部组合。

　　15.

![](http://images.cnitblog.com/news/331782/201305/02140958-0ff0d174ed79413d9052797fab044596.png)

　　"部分匹配值"就是"前缀"和"后缀"的最长的共有元素的长度。以"ABCDABD"为例，

> －　"A"的前缀和后缀都为空集，共有元素的长度为0；
>
> 　　－　"AB"的前缀为\[A\]，后缀为\[B\]，共有元素的长度为0；
>
> 　　－　"ABC"的前缀为\[A, AB\]，后缀为\[BC, C\]，共有元素的长度0；
>
> 　　－　"ABCD"的前缀为\[A, AB, ABC\]，后缀为\[BCD, CD, D\]，共有元素的长度为0；
>
> 　　－　"ABCDA"的前缀为\[A, AB, ABC, ABCD\]，后缀为\[BCDA, CDA, DA, A\]，共有元素为"A"，长度为1；
>
> 　　－　"ABCDAB"的前缀为\[A, AB, ABC, ABCD, ABCDA\]，后缀为\[BCDAB, CDAB, DAB, AB, B\]，共有元素为"AB"，长度为2；
>
> 　　－　"ABCDABD"的前缀为\[A, AB, ABC, ABCD, ABCDA, ABCDAB\]，后缀为\[BCDABD, CDABD, DABD, ABD, BD, D\]，共有元素的长度为0。

　　16.

![](http://images.cnitblog.com/news/331782/201305/02140958-eb88ca7ad6d54fe4b44dc9ec0061f745.png)

　　"部分匹配"的实质是，有时候，字符串头部和尾部会有重复。比如，"ABCDAB"之中有两个"AB"，那么它的"部分匹配值"就是2（"AB"的长度）。搜索词移动的时候，第一个"AB"向后移动4位（字符串长度-部分匹配值），就可以来到第二个"AB"的位置。

**　　算法时间复杂度为O\(m+n\)（其中m为字符段长度，n为匹配模式的长度）。**





### 3 算法实现

void getNext\(const std::string &p, std::vector&lt;int&gt; &next\)

{

    next.resize\(p.size\(\)\);

    next\[0\] = -1;



    int i = 0, j = -1;

    

    while \(i != p.size\(\) - 1\)

    {

        //这里注意，i==0的时候实际上求的是next\[1\]的值，以此类推

        if \(j == -1 \|\| p\[i\] == p\[j\]\)

        {

            ++i;

            ++j;

            next\[i\] = j;

        }

        else

        {

            j = next\[j\];

        }

    }

}



int kmp\(const std::string& s, const std::string& p, const int sIndex = 0\)

{

    std::vector&lt;int&gt;next\(p.size\(\)\);

    getNext\(p, next\);//获取next数组，保存到vector中



    int i = sIndex, j = 0;

    while\(i != s.length\(\) && j != p.length\(\)\)

    {

        if \(j == -1 \|\| s\[i\] == p\[j\]\)

        {

            ++i;

            ++j;

        }

        else

        {

            j = next\[j\];

        }

    }



    return j == p.length\(\) ? i - j: -1;

}

```

```



