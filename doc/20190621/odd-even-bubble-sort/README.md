# 冒泡排序 - 数组中的奇数放在偶数之前

## 问题描述

输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间是从小到大排序的。

## 分析
传统的排序规则是，如果前面一个数大于后面一个数，则交换位置。这道题目增加了一个条件，就是指定奇数在前，偶数在后，那么根据题意，只要抓住以下关键点来判断：

1. 如果偶数在奇数前面，则需要交换；
2. 比较两个数字，如果都为奇数或者都为偶数的情况下，前者如果大于后者，那么需要交换。



## 代码


第一段代码：
``` java 
  public static int[] bubbleSort1(int[] arr) {
    for (int i = 0; i < arr.length - 1; i++) {
      for (int j = arr.length - 1; j > i; j--) {//-1为了防止溢出
        if ((arr[j] % 2 == arr[j - 1] % 2 && arr[j] < arr[j - 1]) || 
        (arr[j - 1] % 2 == 0 && arr[j] % 2 == 1)) {
          int temp = arr[j];
          arr[j] = arr[j - 1];
          arr[j - 1] = temp;
        }
      }
    }
    return arr;
  }
```

第二段代码：

``` java 
  public static int[] bubbleSort2(int[] arr) {
    for (int i = 0; i < arr.length - 1; i++) {
      for (int j = 0; j < arr.length - i - 1; j++) {//-1为了防止溢出
        if ((arr[j] % 2 == arr[j + 1] % 2 && arr[j] > arr[j + 1]) ||
            (arr[j + 1] % 2 == 1 && arr[j] % 2 == 0)) {
          int temp = arr[j];
          arr[j] = arr[j + 1];
          arr[j + 1] = temp;

        }
      }
    }
    return arr;
  }
```

两段代码的不同点为内层循环 j 的遍历顺序：第一段代码中，j 是由高位遍历到低位，因此判断的时候的时候会跟 j - 1 位比较；第二段代码 j 则是由低往高遍历的，那么判断的时候与 j + 1 的位置进行判断。两者的代码输出的遍历顺序不同，但是输出的结果是相同。

