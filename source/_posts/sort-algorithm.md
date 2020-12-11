---
title: 经典排序算法
date: 2020-03-11 14:00:34
categories:
    - Algorithm
tags:
    - 排序
    - todo
---

### 常用排序算法

已收录：冒泡排序、简单选择排序、简单插入排序、二路归并排序、简单快速排序

<!--more-->

常用排序算法，要掌握的要点：
- 实现思路
- 时间复杂度
- 最优时间复杂度，最差时间复杂度

#### 冒泡排序

冒泡排序是重复遍历元素，对元素进行两两比较，如果遇到顺序错误就进行调换，直到所有元素顺序都没有错误

>思路：
>1. 相邻元素比较，比如第一个元素和第二个元素比较，大的元素移动到后面
>2. 继续比较每一对相邻元素，大的元素放后面，一直到最后一对元素比较完毕，最大的元素就到最后面了
>3. 重复上述步骤，再比较时不必比较上一次循环确定的最大元素也就是最后一个元素
>4. 重复1-3步骤，直到全部比较完毕

```java
    public int[] bubbleSort(int[] arrays) {
        if (arrays == null || arrays.length < 2) {
            return arrays;
        }

        for (int i = 0; i < arrays.length; i++) {
            for (int j = 0; j < arrays.length - 1 - i; j++) {
                if (arrays[j] > arrays[j + 1]) {
                    int temp = arrays[j + 1];
                    arrays[j + 1] = arrays[j];
                    arrays[j] = temp;
                }
            }
        }
        return arrays;
    }

```

时间复杂度：O(n^2)


#### 选择排序

简单选择排序也是重复遍历序列。开始未排序序列找到最小元素，放到起始排序位置，再从剩余未排序序列继续查找最小元素，放到已排序序列末尾

>思路：
>1. 拿每一个跟后续所有元素比较，找到最小元素
>2. 最小元素和当前所在元素交换位置
>3. 重复1、2步骤直到所有元素都遍历了一遍

```java
    public int[] selectionSort(int[] arrays) {
        if (arrays == null || arrays.length < 2) {
            return arrays;
        }

        int length = arrays.length;

        int minIndex;

        for (int i = 0; i < length; i++) {
            minIndex = i;
            //这里使用i+1，因为每次遍历之后第一项肯定是最小的，否则会引入脏数据
            for (int j = i + 1; j < length; j++) {
                if (arrays[j] < arrays[minIndex]) {
                    minIndex = j;
                }
            }
            int temp = arrays[i];
            arrays[i] = arrays[minIndex];
            arrays[minIndex] = temp;
        }

        return arrays;
    }
```

所用时间复杂度也是O(n^2)

#### 插入排序法

插入排序法，从最小子序列开始构建有序序列，每次从未排序序列往子序列加入一个元素，就对该元素进行排序。直到子序列和目标序列一样大

>思路：
>1. 从第二个元素开始遍历数组，每一轮拿当前位置元素跟前面所有元素比较
>2. 比较过程中，比当前元素大的后移一位，直到遇到第一个比当前元素小的元素
>3. 将当前元素插入到小元素后面一位
>4. 重复1-3步骤直到全部遍历完毕

```java
    public int[] insertionSort(int[] arrays) {
        int length = arrays.length;
        int preIndex, currValue;
        for (int i = 1; i < length; i++) {
            preIndex = i - 1;
            currValue = arrays[i];
            while (preIndex >= 0 && arrays[preIndex] > currValue) {
                arrays[preIndex + 1] = arrays[preIndex];
                preIndex--;
            }
            arrays[preIndex + 1] = currValue;
        }
        return arrays;
    }
```

所用时间复杂度也是O(n^2)

#### 归并排序（二路归并排序）

归并排序采用的是分治的思想，将已有序子序列合并，得到完全有序序列，因此需要先拿到有序子序列，让子序列在段间有序。常用的是二路归并排序。

实现思路：
>1. 将长度为n的序列拆分为两个长度为n/2的序列
>2. 对子序列进行1的操作，找到最小序列，开始向上归并
>3. 归并过程中对子序列进行排序，最终拿到两段有序子序列
>4. 最终两个子序列合并排序，形成完全有序序列

```java
    public int[] mergeSort(int[] arrays) {
        if (arrays.length < 2) {
            return arrays;
        }
        int n = arrays.length / 2;
        int[] left = new int[n];
        int[] right = new int[arrays.length - n];
        System.arraycopy(arrays,0,left,0,left.length);
        System.arraycopy(arrays,n,right,0,right.length);
        return mergeArrays(mergeSort(left), mergeSort(right));
    }

    /**
     * 对左右数组进行合并排序
     */
    public int[] mergeArrays(int[] left, int[] right) {
        int[] mergeArrays = new int[left.length + right.length];
        int a = 0, b = 0, index = 0;
        for (int i = 0; i < mergeArrays.length; i++) {
            if (a >= left.length) {
                mergeArrays[i] = right[b];
                b++;
            } else if (b >= right.length) {
                mergeArrays[i] = left[a];
                a++;
            } else {
                if (left[a] < right[b]) {
                    mergeArrays[i] = left[a];
                    a++;
                } else {
                    mergeArrays[i] = right[b];
                    b++;
                }
            }
        }
        return mergeArrays;
    }
```

归并排序的时间复杂度分析：

1. 看mergeArrays函数，时间复杂度是O(n)
2. 再看mergeSort函数，虽然使用了递归，但是核心是分治，也就是把n进行k次对半拆分，也就是 2^k = n,所以时间复杂度就是log n
3. 综合1和2可以得到，函数整体时间复杂度是 O(log n)
4. 因为无论序列中元素是否有序，都要进行分治，所以最好时间复杂度和最坏时间复杂度应该是一致的

稳定性分析：
因为两个相同元素不会改变先后位置，所以是稳定的。

总结：
- 平均时间复杂度 O(nlog n)
- 最好和最差时间复杂度都是 O(nlog n)
- 二路归并排序属于稳定排序

#### 快速排序

快速排序：选定一个基准，经过一次排序将序列分为两个子序列，一部分比基准小，一部分比基准大。然后对子序列进行同样操作，最终实现序列有序。

实现思路：
1. 选定第一个元素作为基准元素，使用两个指针，分别指向第一个元素和最后一个元素
2. 先用右边的指针与基准比较，直到遇见第一个比基准小的元素停止(先不考虑指针相遇)
3. 再用左边元素与基准元素比较，直到遇见比基准大的元素停止(先不考虑相遇情况)
4. 将左右元素此时指向的元素调换位置
5. 继续重复2~4步骤，直到指针相遇。注意2、3步骤有可能出现没有符合条件直接相遇的情况。这两种相遇情况下，都把此时相遇为止元素 同基准元素调换位置
6. 此时基准元素作为分割点，左边都是小元素序列，右边都是大元素序列。分别对左右序列(不包括基准元素)，递归执行1~5步骤，直到分割后左右指针相遇为止。

上代码：
```java
    public int[] quickSort(int[] arrays) {
        return quickSort(arrays, 0, arrays.length - 1);
    }

    private int[] quickSort(int[] arrays, int left, int right) {
        if (left < right) {
            //标定基准，同时以基准对arrays left-right进行快速排序
            int partition = partition(arrays, left, right);
            //对基准左边数组进行快排
            quickSort(arrays, left, partition - 1);
            //对基准右边数组进行快排
            quickSort(arrays, partition + 1, right);

        }
        return arrays;
    }

    private int partition(int[] arrays, int left, int right) {
        int pivot = arrays[left];//锚定基准点
        int j = right;
        int i = left;
        while (i < j) {
            // 从右侧开始检查，找到第一个小于基数的元素位置
            while (i < j && arrays[j] >= pivot) {
                j--;
            }

            //从左侧开始找，找到第一个大于基数的元素位置
            while (i < j && arrays[i] <= pivot) {
                i++;
            }

            //只有位置不同的时候再交换
            if (i != j) {
                swap(arrays, i, j);
            }

        }
        //同一位置没必要交换
        if (left != i) {
            swap(arrays, left, i);
        }

        return i;
    }

    private int[] swap(int[] array, int i, int j) {
        int temp = array[i];
        array[i] = array[j];
        array[j] = temp;
        return array;
    }
```

时间复杂度：

结论：最好时间复杂度是O(1),最差时间复杂度是O(n^2),平均时间复杂度是O(nlog n)
分析：
- n = 1 的时候，直接就返回了，O(1)
- n > 1
    - 假设每次序列都是恰好从中间分割，那n可以分2^k层，也就是log n，而每一层不管分多少子序列，加起来都是遍历n次。log n 层 n次遍历，可以得到平均时间复杂度O(nlog n)
    - 最差的情况就是，每次基准点都是在序列一边，导致分了n层，每一层遍历n次，时间复杂度O(n^2)


#### 堆排序

堆排序（Heapsort）：利用堆数据结构设计的排序算法

tips：关于堆排序使用时仍然可以利用数组，但是要在脑海中构建出堆的数据结构模型。而构建的模型跟二叉树是一样的，所以可以借用二叉树的一些概念

实现堆排序必须知道的：
- 确定堆中第一个非叶子节点：length/2 -1
- 针对每个非叶子节点，他的左右子节点都是 2i+1 和 2i+2
- 需要知道堆的两种模型：大顶堆和小顶堆
    - 大顶堆：每个节点都大于等于它的左右节点的堆
    - 小顶堆：每个节点都小于等于它的左右孩子节点的堆
    
tips： 大顶堆一般用于构建升序序列，小顶堆用于构建降序序列。

了解了以上节点，可以来看一下实现思路
1. 首先需要从第一个非叶子节点开始遍历数组，从下到上，从右到左，构建大顶堆
2. 此时大顶堆根节点就是最大元素，将他与最后一个叶子节点交换位置。
3. 排除最后一个叶子节点，重新调整大顶堆
4. 重复步骤2-3直到还剩一个元素为止。此时序列就已经是有序并且升序的了，


```java
    public int[] heapSort(int[] arrays) {

        int length = arrays.length;
        //首先构建大顶堆
        for (int i = length / 2 - 1; i >= 0; i--) {
            //找到第一个非叶子节点，下到上，右到左。
            // 可以自行画图找规律 length / 2 - 1
            adjustHeap(arrays, i, length);
        }

        //每次把堆顶元素与末尾元素交换，堆减少一个元素，直到遍历完成
        for (int i = length - 1; i > 0; i--) {
            //交换堆顶元素,与末尾元素
            swap(arrays, 0, i);
            //重新构建大顶堆
            adjustHeap(arrays, 0, i);
        }

        return arrays;
    }

    /**
     * 堆排序核心 - 调整 大顶堆
     */
    public void adjustHeap(int[] arrays, int i, int length) {
        int temp = arrays[i];//临时存储 i 对应元素
        //每次选取指定节点的左节点进行比较
        for (int k = 2 * i + 1; k < length; k = 2 * k + 1) {
            //比较左右节点大小，右节点大选择右节点作为比较对象
            if (k + 1 < length && arrays[k] < arrays[k + 1]) {
                k++;
            }

            //比较和temp元素的大小, 若temp小则被换掉
            // 因为每次比较的都是temp，k位置元素不用改变，最后再改变即可
            if (arrays[k] > temp) {
                arrays[i] = arrays[k];
                i = k;//i直接用来存储每次交换后新的k的位置
            }

        }

        //最终i的值确定以后把临时temp存进去，完成大顶堆的调整
        arrays[i] = temp;

    }

    private void swap(int[] array, int i, int j) {
        int temp = array[i];
        array[i] = array[j];
        array[j] = temp;
    }

```

关于时间复杂度：堆排序的时间复杂度三种情况都是O(nLog n)

1. 首先分析核心代码 adjustHeap 中的时间复杂度 2的m次方等于n，执行次数也就是 （log n） 次
2. 然后分析heapSort 两个for循环实际上是 nLog n + nLog n 
综上时间复杂度是 O(nLog n)


#### 计数排序&桶排序

计数排序是一种特殊的桶排序，他们的思想是一致的，都是借助开辟新的内存空间进行排序

##### 计数排序

先来看简单的计数排序实现思路

1. 遍历原数组，找出最大和最小元素，根据最大和最小元素创建 计数数组 bucketArray 容量，相当于min、min+1、min+2··· max 所有元素都有对应位置
2. 遍历原数组，记录每一个元素出现次数，存入 bucketArray 中，这里有一个前提条件，在存入次数的时候实际上已经把顺序确定了
3. 创建新的数组 dstArray，用于存放排序后的元素
4. 此时如果把bucketArray中次数大于0的位置，找到srcArrays中对应的值，依次插入 dstArray n次就是有序序列了，下面转换成代码思路
    1. 遍历 bucketArray 数组，重新计算每一项的值等于它的值加上前一项的值，这就意味着按照步骤3的思路，任意值在 dstArray 中的最大可能位置就确定了。
    2. 遍历原数组，根据 bucketArray[i]确定插入位置，之后 bucketArray[i]-- , 下次重复数据就会插入到后一个位置



```java
    
    public int[] countingSort(int[] arrays) {
        if (arrays == null || arrays.length <= 1) {
            return arrays;
        }
        int min = arrays[0], max = arrays[0];
        int srcLength = arrays.length;
        //遍历找到最大值和最小值
        for (int temp : arrays) {
            max = Math.max(temp, max);
            min = Math.min(temp, min);
        }

        //构建计数数组
        int bucketLength = max - min + 1;
        int[] bucketArray = new int[bucketLength];

        //再次遍历原数组，找出元素重复数量
        for (int array : arrays) {
            bucketArray[array - min]++;
        }

        //每一项与前一项叠加
        for (int i = 1; i < bucketLength; i++) {
            bucketArray[i] += bucketArray[i - 1];
        }

        //开始再次遍历原数组
        int[] dstArray = new int[srcLength];
        for (int i = srcLength - 1; i >= 0; i--) {
            int target = arrays[i];
            //计算得到当前元素之前又多少个元素
            int bucketIndex = target - min;
            int index = bucketArray[bucketIndex] - 1;
            dstArray[index] = target;
            bucketArray[bucketIndex]--;
        }
        return dstArray;
    }
```

计数排序的时间复杂度是O(n),但是他更适用于序列不大比较集中的情况。因为要开辟额外的存储空间，所以数据比较分散的时候就不适合了，会浪费空间

##### 桶排序

桶排序也需要开辟新的存储空间，可以使用链表型数据结构来实现，java里更常用的是ArrayList

实现思路：
1. 找到序列最大最小值，将最大最小值之间的范围进行 n 等分，这个可以自行决定，一般来说n越大排序越快，但是消耗空间更多
2. 遍历序列，计算应该插入桶中哪个位置
3. 遍历桶，每一项中元素进行排序
4. 将桶中所有子序列取出来，拼接起来就是有序序列

```java
     public void bucketSort(int[] arrays) {
        int min = arrays[0], max = arrays[0];
        int srcLength = arrays.length;
        //遍历找到最大值和最小值
        for (int temp : arrays) {
            max = Math.max(temp, max);
            min = Math.min(temp, min);
        }
        //也可以使用别的分组方式
        int bucketSize = (max - min) / srcLength + 1;

        //构建桶
        List<ArrayList<Integer>> bucketList = new ArrayList<>();
        for (int i = 0; i < bucketSize; i++) {
            bucketList.add(new ArrayList<>());
        }

        for (int array : arrays) {
            int index = (array - min) / srcLength;
            bucketList.get(index).add(array);
        }

        //对桶中元素进行排序
        for (ArrayList<Integer> item : bucketList) {
            if (!item.isEmpty()){
                //使用官方API，也可以根据需要使用别的排序法
                Collections.sort(item);
            }
        }

        System.out.println("bucket arrays: " + bucketList.toString());
    }
```

时间复杂度分析：

最好情况：O(n)，一般n分的越多时间复杂度越低，但是空间消耗越高

如果n设定合理，时间复杂度就取决于桶中子序列排序的时间复杂度了应该是 n * O(f(n))


### 未完待续

### 待补充
由于理解不透彻暂不记录，但是标记以防以往。
- 希尔排序
- 快速排序的多种思路
- 空间复杂度

#### 参考文献

1. [十大经典排序算法（动图演示）](https://www.cnblogs.com/onepixel/p/7674659.html)
2. [图解排序算法-希尔排序](https://www.cnblogs.com/chengxiao/p/6104371.html)
3. [图解排序算法-堆排序](https://www.cnblogs.com/chengxiao/p/6129630.html)

#### Log

- 20200321 新增堆排序
- 20200314 新增快速排序
- 20200311 创建本文，新增冒泡排序、选择排序、插入排序、归并排序