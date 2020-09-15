## 归并排序



### 是什么？

归并排序（MERGE-SORT）是建立在归并操作上的一种有效的排序算法,该算法是采用**分治法**（Divide and Conquer）的一个非常典型的应用。将已有序的子序列合并，得到完全有序的序列；即先使每个子序列有序，再使子序列段间有序。**若将两个有序表合并成一个有序表，称为二路归并。**



### 归并过程

比较a[i]和b[j]的大小，若a[i]≤b[j]，则将第一个有序表中的元素a[i]复制到r[k]中，并令i和k分别加上1；否则将第二个有序表中的元素b[j]复制到r[k]中，并令j和k分别加上1，如此循环下去，直到其中一个有序表取完，然后再将另一个有序表中剩余的元素复制到r中从下标k到下标t的单元。归并排序的算法我们通常用递归实现，先把待排序区间[s,t]以中点二分，接着把左边子区间排序，再把右边子区间排序，最后把左区间和右区间用一次归并操作合并成有序的区间[s,t]。

### 步骤

1. 申请空间，使其大小为两个已经排序序列之和，该空间用来存放合这两个并后的序列
2. 设定两个指针，最初位置分别为两个已经排序序列的起始位置
3. 比较两个指针所指向的元素，选择相对小的元素放入元素组对应位置，并移动指正到下一位置
4. 重复步骤3直到某一指针超出序列尾
5. 将另一序列剩下的所有元素直接复制到合并序列尾



#### 总结

* 将**两个**已**排好序**的数组**合并**成**一个有序**的数组,称之为归并排序
* 步骤：遍历两个数组，比较它们的值。谁比较小，谁先放入大数组中，直到数组遍历完成

```java
public class MergeSort {


    public static void main(String[] args) {
        int[] arrays = {9, 2, 5, 1, 3, 2, 9, 5, 2, 1, 8};
        mergeSort(arrays, 0, arrays.length - 1);
        System.out.println(Arrays.toString(arrays));
    }

    public static void mergeSort(int[] arrays, int begin, int end) {
        if (begin == end) {
            return;
        }

        int mid = (end + begin) / 2;
        // 左划分一块
        mergeSort(arrays, begin, mid);
        // 右划分一块
        mergeSort(arrays, mid + 1, end);

        merge(arrays, begin, mid, end);

    }

    private static void merge(int[] arrays, int begin, int mid, int end) {

        int[] back = Arrays.copyOfRange(arrays, begin, end + 1);

        int i = begin;
        int j = mid + 1;
        for (int k = begin; k <= end; k++) {
            if (i > mid) {
                arrays[k] = back[j++ - begin];
            } else if (j > end) {
                arrays[k] = back[i++ - begin];
            } else if (back[i - begin] < back[j - begin]) {
                arrays[k] = back[i++ - begin];
            } else {
                arrays[k] = back[j++ - begin];
            }
        }

    }
}
```









## 参考

https://juejin.im/post/5ab4c7566fb9a028cb2d9126

