下面举例说明：

1. 给定一个整形数组a\[\]={16,7,3,20,17,8}，对其进行堆排序。

   首先根据该数组元素构建一个完全二叉树，得到

   [![](http://e.hiphotos.baidu.com/exp/w=500/sign=8b3084880855b3199cf9827573ab8286/1c950a7b02087bf42e229f60f0d3572c10dfcf69.jpg "堆排序算法解析")](http://jingyan.baidu.com/album/5225f26b057d5de6fa0908f3.html?picindex=1)

2. 然后需要构造初始堆，则从最后一个非叶节点开始调整，调整过程如下：

   [![](http://a.hiphotos.baidu.com/exp/w=500/sign=9f23bd108026cffc692abfb289004a7d/63d9f2d3572c11df818a74aa612762d0f603c2e4.jpg "堆排序算法解析")](http://jingyan.baidu.com/album/5225f26b057d5de6fa0908f3.html?picindex=2)

3. 20和16交换后导致16不满足堆的性质，因此需重新调整

   [![](http://h.hiphotos.baidu.com/exp/w=500/sign=55ec2eaf0b24ab18e016e13705fbe69a/4b90f603738da9773e984da2b251f8198718e380.jpg "堆排序算法解析")](http://jingyan.baidu.com/album/5225f26b057d5de6fa0908f3.html?picindex=3)

4. 这样就得到了初始堆。

   即每次调整都是从父节点、左孩子节点、右孩子节点三者中选择最大者跟父节点进行交换\(交换之后可能造成被交换的孩子节点不满足堆的性质，因此每次交换之后要重新对被交换的孩子节点进行调整\)。有了初始堆之后就可以进行排序了。

   [![](http://h.hiphotos.baidu.com/exp/w=500/sign=36ca538d5b82b2b7a79f39c401accb0a/95eef01f3a292df57770679cbe315c6035a8738c.jpg "堆排序算法解析")](http://jingyan.baidu.com/album/5225f26b057d5de6fa0908f3.html?picindex=4)

5. 此时3位于堆顶不满堆的性质，则需调整继续调整

   [![](http://c.hiphotos.baidu.com/exp/w=500/sign=2471e32c087b02080cc93fe152d8f25f/f7246b600c338744a18fe4f5530fd9f9d62aa0f5.jpg "堆排序算法解析")](http://jingyan.baidu.com/album/5225f26b057d5de6fa0908f3.html?picindex=5)  
   [![](http://h.hiphotos.baidu.com/exp/w=500/sign=9e7ddf86ba0e7bec23da03e11f2fb9fa/9358d109b3de9c82762432066e81800a18d843f2.jpg "堆排序算法解析")](http://jingyan.baidu.com/album/5225f26b057d5de6fa0908f3.html?picindex=6)

6. 7  
    这样整个区间便已经有序了。

```java
    public void heapSort(int[] a) {
        int i;
        for (i = a.length / 2 - 1; i >= 0; i--) {// 构建一个大顶堆
            this.adjustHeap(a, i, a.length);
            this.printlist(a);
        }
        System.out.println("finish build heap");
        for (i = a.length-1; i >= 0; i--) {// 将堆顶记录和当前未经排序子序列的最后一个记录交换
            int temp = a[0];
            a[0] = a[i];
            a[i] = temp;
            this.adjustHeap(a, 0, i);// 将a中前i-1个记录重新调整为大顶堆
            this.printlist(a);
        }
    }
    
    /**
     * 构建大顶堆
     */
    private void adjustHeap(int[] a, int i, int len) {
        int temp, j;
        temp = a[i];
        for (j = 2 * i+1; j < len; j *= 2) {// 沿关键字较大的孩子结点向下筛选
            if (j+1 < len && a[j] < a[j + 1])
                ++j; // j为关键字中较大记录的下标
            if (temp >= a[j])
                break;
            a[i] = a[j];
            i = j;
        }
        a[i] = temp;
    }
```



