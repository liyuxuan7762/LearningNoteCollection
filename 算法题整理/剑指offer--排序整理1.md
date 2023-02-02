### JZ53 数字在升序数组中出现的次数

描述

给定一个长度为 n 的非降序数组和一个非负数整数 k ，要求统计 k 在数组中出现的次数

**解题思路1**：可以利用二分法查找。利用二分法查找到元素的位置。因为是升序数组，就说明只要找到给定元素，然后再判断元素左边和右边是否有和这个元素相等的元素即可。

```java
public int GetNumberOfK(int[] array, int k) {
    if (array == null || array.length == 0) return 0;

    int count = 0;
    int left = 0, right = array.length - 1;

    while (left <= right) {
        int mid = (left + right) / 2;
        if (array[mid] == k) {
            count++;
            // 遍历两边元素 看看有没有相等的
            for (int i = mid; i >=0 ; i--) {
                if (array[i] != array[mid])
                    break;
                count++;
            }
            for (int i = mid; i < array.length ; i++) {
                if (array[i] != array[mid])
                    break;
                count++;
            }
        } else if (array[mid] < k) {
            // 到右边去找
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return count;
}
```

> 采用上面的这种方式比较好理解，但是如果一个数组中target的值过多，就会使二分查找退化为On时间复杂度

**解题思路2:**依然是利用二分法，查找到第一个等于target的元素下面和最后一个等于target的元素下标，两个下标相减+1就是target出现的次数。

```java
public int GetNumberOfK(int[] array, int k) {
    if (array == null || array.length == 0) return 0;

    int left = equalAndGreater(array, k);
    int right = greater(array, k) - 1;
    System.out.println(left);
    System.out.println(right);

    if (left < 0 || right < 0) return 0;

    return right - left + 1;
}

// 查找第一个大于等于target的下标 === 查找第一个等于target的下标
public int equalAndGreater(int[] array, int k) {
    if (array == null || array.length == 0) return -1;

    int left = 0, right = array.length - 1;

    while (left <= right) {
      int mid = (right + left) / 2;
      if (array[mid] < k) {
        left = mid + 1;
      } else {
        right = mid - 1;
      }
    }
    return left;
}
// 查找第一个大于target的下标-1 === target的最后下标
public int greater(int[] array, int k) {
    if (array == null || array.length == 0) return -1;
    int left = 0, right = array.length - 1;
    while (left <= right) {
      int mid = (right + left) / 2;
      if (array[mid] <= k) {
        left = mid + 1;
      } else {
        right = mid - 1;
      }
    }
    return left;
}
```

> 下面整理了二分法中常见的一些变种问题

**求第一个大于等于target的下标（等价于求第一个等于target的下标）**

```java
// 查找第一个大于等于target的下标 === 查找第一个等于target的下标
public int equalAndGreater(int[] array, int k) {
    if (array == null || array.length == 0) return -1;

    int left = 0, right = array.length - 1;

    while (left <= right) {
      int mid = (right + left) / 2;
      if (array[mid] < k) {
        left = mid + 1;
      } else {
        right = mid - 1;
      }
    }
    return left;
}
```

**求第一个大于target的下标（等价于求最后一个target元素下标+1）**

```java
// 查找第一个大于target的下标-1 === target的最后下标
public int greater(int[] array, int k) {
    if (array == null || array.length == 0) return -1;
    int left = 0, right = array.length - 1;
    while (left <= right) {
      int mid = (right + left) / 2;
      if (array[mid] <= k) {
        left = mid + 1;
      } else {
        right = mid - 1;
      }
    }
    return left;
}
```

**查找第一个小于target的值的下标**

```java
public int binarySearchOne() {
   int[] array = {0,1,2,3,4,5,6};
   int target = 3;
	int left = 0, right = array.length-1;
	while(left <= right) {
	    int mid = (left + right)/2;
    //只修改了判断的条件，相当于将大于等于归为一类。
		if(array[mid] >=  target)
            right = mid - 1;
		else
		     left = mid + 1;
	}
	return right;
}
```

**查找第一个小于等于target值的下标**

```java
public int binarySearchOne() {
   int[] array = {0,1,2,3,4,5,6};
   int target = 3;
	int left = 0, right = array.length-1;
	while(left <= right) {
	    int mid = (left + right)/2;
    //只修改了判断的条件，相当于将大于等于归为一类。
		if(array[mid] >  target)
            right = mid - 1;
		else
		     left = mid + 1;
	}
	return right;
}
```

> 我们可以看出来，这几个代码实际修改的地方就只有array[mid]和target的判断条件。实际上，这个条件和要求的条件正好为补集。
>
> * 求大于target的，就是array[mid]小于等于target
> * 求大于等于target的，就是array[mid]小于target
> * 求小于target的，就是array[mid]大于等于target
> * 求小于等于target的，就是array[mid]大于target
>
> 相关博客资料https://blog.csdn.net/qq_29611345/article/details/103080287

### JZ11 旋转数组的最小数字

描述

有一个长度为 n 的非降序数组，比如[1,2,3,4,5]，将它进行旋转，即把一个数组最开始的若干个元素搬到数组的末尾，变成一个旋转数组，比如变成了[3,4,5,1,2]，或者[4,5,1,2,3]这样的。请问，给定这样一个旋转数组，求数组中的最小值。

```java
public int minNumberInRotateArray(int [] array) {
    if (array == null || array.length == 0) return 0;
    int left = 0, right = array.length - 1;
    while (left < right) {
        int mid = (left + right) / 2;
        if (array[mid] < array[right]) {
          // mid落在右边
          right = mid;
        } else if (array[mid] > array[right]) {
          // mid落在左边
          left = mid + 1;
        } else {
          right--;
        }
    }
    return array[left];
}
```

> 因为旋转点一定会落在右边的子区间中，因此当mid落入右边的区间时，使用的是right = mid 而不是 right = mid - 1
>
> 原因是mid也可能是连接点，如下面的区间
>
> [5, 6, 7, 1, 2, 3, 4] 
>
> left = 0, right = 6, mid = 3, array[mid] = 1;
>
> 此时 array[mid] < mid[right] = 4
>
> 因此mid落在了右边区间中，而且我们可直观看到，mid就是旋转点。如果这个时候执行right = mid - 1，那么旋转点就被忽略了。

**解题思路2:**数组的旋转将原数组分割为两个字数组A，B。并且A和B都是单调不减的。并且，序列B中的元素要小于序列A中元素。

* 那么我们可以使用两个指针，指向数组的第一个和第二个元素。使用循环，循环结束条件为fast = array.length

* 判断array[slow] > array[fast]，则说明到了旋转点，且array[fast]就是旋转点，则直接返回

* 如果整个循环结束了，都没有返回，那么说明这个数组就是一个有序数组，那么数组的第一个元素为最小值

```java
public int minNumberInRotateArray(int [] array) {
    // 如果数组为空
    if (array == null || array.length == 0) return 0;
    // 如果数组中只有一个元素
    if (array.length == 1) return array[0];
    int slow = 0, fast = 1;
    while (fast < array.length) {
        if (array[slow] > array[fast])
          	return array[fast];
        slow++;
        fast++;
    }
    return array[0];
}
```

