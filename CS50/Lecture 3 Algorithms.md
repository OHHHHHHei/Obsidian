# 运行速度

1. 用O表示最慢运行时间，如：O($n^2$)
2. 用$\Omega$表示最快运行时间
3. 当最快运行时间与最慢运行时间相等时，使用$\Theta$表示运行速度

# 结构体

1.定义一个结构体

```C
typedef struct
{
    string name;
    string number;
} person;
```

这表示我们创建了一个数据类型：person，其中包含了字符串name与number。

```C
person[0].name = "一个名字"
person[0].number = "一个电话号码"
```

这表示定义person[0]位置的name以及number。

# 排序

## 1.Selection sort（选择排序）

其核心思想是：在每一轮中，从未排序的部分选出最小（或最大）的元素，将其放到已排序部分的末尾，通过不断地选择和交换，逐步构建出有序序列。  
我们可以写出伪代码  

```C
For i from 0 to n–1
    Find smallest number between numbers[i] and numbers[n-1]
    Swap smallest number with numbers[i]
```

写出代码如下
```C
#include <stdio.h>

void selectionSort(int arr[], int n) {
    int i, j, min_idx;

    // 外层循环：每次为“已排序区”增加一个元素
    for (i = 0; i < n - 1; i++) {
        // --- 这就是教练找“冠军”的过程 ---
        // 假设当前“未排序区”的第一个就是最矮的
        min_idx = i; 
        
        // 遍历后面所有的人，找一个更矮的
        for (j = i + 1; j < n; j++) {
            if (arr[j] < arr[min_idx]) {
                min_idx = j; // 找到了更矮的，记住他的位置
            }
        }
        
        // --- 交换位置 ---
        // 让找到的最矮的人和当前“未排序区”的第一个人交换
        int temp = arr[min_idx];
        arr[min_idx] = arr[i];
        arr[i] = temp;
    }
}
```


显然我们我们要重复 $(n - 1) + (n - 2) + … + 1$次，等差数列求和可得总数为 $(n^2-n)/2$

我们需要进行O（ $n^2$）

## 2.Bubble sort（冒泡排序）

这是一种简单的排序算法。它重复地遍历要排序的数列，一次比较两个元素，如果它们的顺序错误就把它们交换过来。遍历数列的工作是重复地进行直到没有再需要交换，也就是说该数列已经排序完成。这个算法的名字由来是因为越小的元素会经由交换慢慢“浮”到数列的顶端。

我们可以写出伪代码

```C
Repeat n - 1 times
    For i from 0 to n – 2
        If numbers[i] and numbers[i + 1] out of order
            Swap them
    If no swaps
        Quit
```

我们可以写出代码如下
```C
#include <stdio.h>

void bubbleSort(int arr[], int n) {
    int i, j;
    // 外层循环：控制总共要“冒泡”多少轮
    for (i = 0; i < n - 1; i++) {
        int swapped = 0; // 优化：标记本轮是否发生交换
        
        // --- 这就是一轮“冒泡”的过程 ---
        // 内层循环：两两比较，把本轮最大的冒泡到后面
        // n-1-i 是因为每轮过后，末尾的元素就已就位，无需再比较
        for (j = 0; j < n - 1 - i; j++) {
            // 如果左边的比右边的大（顺序不对）
            if (arr[j] > arr[j + 1]) {
                // 就交换位置，像气泡一样“上浮”
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
                swapped = 1; // 发生了交换
            }
        }
        
        // 如果一整轮都没有发生交换，说明已经有序，提前结束
        if (swapped == 0) {
            break;
        }
    }
}
```


我们需要进行 $(n-1) \times(n-1)$次,即为O（ $n^2$）。当然，最好的情况下我们只需要 $\Omega$（ $n$）次。 **[各种排序](https://www.cs.usfca.edu/~galles/visualization/ComparisonSort.html)**

# 递归

在计算机科学中，递归指的是一个函数直接或间接调用自身的过程。

例如搭建如下

```C
  #
  ##
  ###
  ####
```

可以使用递归，先搭建前$n-1$层，再搭建第$n$层。

```C
void draw(int n)
{
    // If nothing to draw
    if (n <= 0)
    {
        return;
    }

    // Draw pyramid of height n - 1
    draw(n - 1);

    // Draw one more row of width n
    for (int i = 0; i < n; i++)
    {
        printf("#");
    }
    printf("\n");
}
```

# Merge sort

利用递归，我们可以进行效率更高的排序。

我们可以写出如下伪代码。

```C
If only one number
    Quit
Else
    Sort left half of number
    Sort right half of number
    Merge sorted halves
```

![[image.png]]

这个算法的运行效率是$\Theta$（ $nlogn$）。