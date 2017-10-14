![](http://images0.cnblogs.com/blog2015/776259/201507/280754329387398.png)



```java
    public void QuickSort(int[] a, int i, int j){
    	int left = i;
    	int right = j;
    	int temp = 0;
    	if(left <= right){
    		temp = a[left];
    		while(left != right){
    			while(right > left && a[right] >= temp)
    				right--;
    			a[left] = a[right];
    			
    			while(left < right && a[left]<=temp)
    				left++;
    			a[right] = a[left];
    		}
    		a[right] = temp;
    		this.QuickSort(a, i, left-1);
    		this.QuickSort(a, right+1, j);
    	}
    }
```



