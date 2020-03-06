---
title: 寒假撸算法之（一）——  几个很easy的排序算法
date: 2020-03-01 15:11:18
categories: 数据结构和算法
tags: [冒泡排序,选择排序,插入排序,对数器]
---

从这篇文章开始，笔者想总结一下寒假所学到的算法，一方面是做个总结，另一方面也是为了接下来的面试做一个预热咯。本文将介绍面试常问的几种排序算法，在其中参杂一些算法题目，文法拙劣还请不吝赐教。



<!-- more -->



## 一、牛刀小试

> 有一个有序数组 A ，无序数组 B ，求 B-A （即 B 中有而 A 中没有的数） ，并打印出来；A 的长度为 M，B 的长度为 N。
>



解法一：用 B 中的每一个数去遍历 A 中的每个数。时间复杂度：T(n) = O(N*M)



解法二：对 B 中的每一个数，在 A 中用二分的方法去寻找。

​			时间复杂度分析：每个数的复杂度 O( logM ) ，总的有 N 个数。故：T(n) = O( N * logM )



解法三：1. 对 B 进行快排（ T(n)=O( N * logN ) ），使 B 变得有序；

​			   2. 采用**外排**的方式，打印在 A 中出现的数。

> 所谓外排，就是哪个数组的数更小，就移动哪个数组的指针进行比较。这里借用一下C/C++的指针，用a，b表示指向数组 A 当前元素和数组 B 当前元素的指针。分为一下三种情况：
>
> $$\begin{cases} 打印 *b , b 后移& \text{*b < *a}\\ 不打印, b后移& \text{*b = *a} \\a后移& \text{*b > *a} \end{cases}$$
>
> 直到 a ，b 有一个到末尾了，即停止算法。

​				时间复杂度分析：T(n) = O( N * logN ) + O( M + N )  【前者是B排序耗费的复杂度，后者要么 a 动，要么 b 动，且不回退，有一个到终点即结束】



此题改用何种解法？

显然，解法一最慢，可以排除。

对于解法二和三，M > N时，用解法三【∵ logM > logN 】；反之，用解法二。





## 二、冒泡排序

思路：将小索引视为水的上层，大索引视为下层，不断缩小排序的范围，从 [0 , N-1 ] ，[ 0 , N-2 ] , ... , 不断减小到[ 0 , 2 ] ，[ 0 , 1 ] ，将大的数沉到最大，小的数浮起到最上面。

实现：外层循环从 N-1 减小到 1 ；内层循环从 0 增加到 end ；不断循环把 [0,end] 中最大的数移动到数组最后。

 ```java
public void BubbleSort(int [] arr){
    if(arr == null || arr.length < 2 )
        return;
    for(int end = arr.length-1 ; end > 0 ; end-- ){  // N ~ 1
     	for(int i = 0 ; i < end ; i++ ){ //[0,N-1],[0,N-2],·····, [0,1]
            if(arr[i] > arr[i+1]){
                swap(arr,i,i+1);
            }
        }   
    }
}

//交换两个数，后文将省略此函数
public void swap(int[] arr , int i ,int j){
    if(arr[i]!=arr[j]){  //使用异或来swap，必须保证两个数的值不相等；
        arr[i] = arr[i] ^ arr[j];
        arr[j] = arr[i] ^ arr[j];
        arr[i] = arr[i] ^ arr[j];
    }
}
 ```

时间复杂度：T(n) = O( $n^2$ );





## 三、选择排序

思路：外层循环从 0 增加到 N-1，控制**选择最小数**的范围 ；内层循环从 i+1 增加到 N-1，从**范围**中找到最小数对应的下标 ；

```java
public void SelectSort(int[] arr){
    if(arr == null || arr.length < 2 )
        return;
    for(int i = 0 ; i < arr.length ; i++ ){// N ~ 1
        int minIndex = i ; 
        for(int j = i+1 ; j < arr.length ; j++ ){ //[0,N-1],[1,N-1],·····, [N-2,N-1]
            minIndex = arr[j] < arr[i] ? j : i ; //minIndex 记录了[i,arr.length]最小值的坐标
        }
        swap(arr,i,minIndex); //arr[i]上保存着[i,N-1]上最小的数
    }
}
```

时间复杂度： T(n) = O( $n^2$ );

无论什么数据进去，都是严格的**T(n)=O(n²)** 。所以用到它的时候，数据规模越小越好。唯一的好处可能就是不占用额外的内存空间了吧。

In fact ，现在冒泡和选择排序在工程上已经没人用了。这里提一下just为了加深对复杂度的理解。



## 四、插入排序

思路：将待排序数组分为**已经排好序**【0 ~ i 】和**未排好序**【i + 1,N-1】两个部分，不断从未排好序的部分抽取数字插入到合适的位置。

类似于打扑克时的插牌，只是此时的插牌动作，表现为：**一直交换，直到前一个数字不比当前数字大**。

```java
public static void insertSort(int[] arr){
    if(arr == null || arr.length < 2)
        return;
    for(int i = 1 ; i < arr.length ; i++ ){
        //for(int j = i - 1 ; j>0 && arr[i]<arr[j] ; j-- ){ //错误的写法,这种写法只会和一个数字比较，而不是相邻的所有数都比较了

        for(int j = i - 1 ; j>0 && arr[j+1]<arr[j] ; j-- ){ //正确的写法  
          swap(arr,j,j+1);
        }
    }
}

```

时间复杂度：插入排序的T(n)和给定序列的状况有关，平均下来 T(n) = O($n^2$) 。

最好情况（已经有序）时，每个数比较 ( O(1) )后， i++ ，加n次，T(n) = O(n) ；

最坏情况（完全逆序）时，每个数比较后需要从最后调整到最前面( O( n ) )，i++，共有n次，T(n) = O($n^2$) 。



## 五、对数器

在介绍下一个算法之前，笔者想问读者大大几个问题，宁是否有过下面蛋疼的烦恼：

- 没有OJ，自己写了题也不知道对不对，不能保证正确；

- 笔试的流程中，小样本（比如说长度为4的数组）过了，但是大样本（比如说长度为4000的数组）错了！而且不知道是大样本中的哪个值导致出错的；
- 自己写了贪心算法，但是无法证明自己的贪心策略是不是正确的。

如果没有，打扰了，请自行滑过这一部分；如果有，请君听我细细道来何为***对数器***。



所谓对数器，就是比对数字的工具。我们可以通过随时随地生成大量的样本来检测自己的算法是否是完备的正确的，而它的使用有如下7个步骤：

1. 有一个待测试的方法a；
2. 实现一个**绝对正确**，但是T(n)可能不是很理想的方法b （**必须保证绝对正确，尽管很慢**）；
3. 实现一个随机样本发生器；
4. 实现**和正确结果比对**的方法；
5. 把方法a和方法b的运算结果对比多次，从而验证a的正确性；
6. 如果有一个样本使得比对出错，打印出该样本，从而为改进算法进行准备；
7. 当样本数量多到一定程度时，自己写的方法a依旧正确 => 可以确定方法a正确。 



以对数组排序为例，构造对数器，代码如下：

```java
//1.绝对正确的方法
public void rightMathod(int[] arr){
    Array.sort(arr);
}


//2.生成随机数组  size：数组大小   value：数组中值的范围   最终范围：[-value,value]
//这个函数需要准备模板（二叉树，堆，数组，排序等等的）！！
public int[] generateRandomArray(int size, int value){
    int[] arr = new int[(int)((size+1)*Math.Random())];
    for(int i = 0 ; i < arr.length ; i++ ){
        arr[i] = (int)((value+1) * Math.Random())-(int)((value+) * Math.Random());
    }
    return arr;
}


//3.实现结果比对    
//这个函数需要准备模板（二叉树，堆，数组，排序等等的）！！
public boolean isEqual(int[] arr1 , int[] arr2){
    if(arr1 == null && arr2 != null  ||  arr1 != null && arr2 == null )
        return false;
    if(arr1 == null && arr2 == null) //都为空，相等
        return true;
    if(arr1.length != arr2.length) //数组不一样长，false
        return false;
    for(int i = 0 ; i < arr1.length ; i++ ){
        if(arr1[i] != arr2[i])
            return false;  //必要时，可在返回前打印出不相等的值的位置
    }
    return true;
}

//4.拷贝值（为了避免直接拿题目给的数组进行操作）
public int[] copyArray(int[] arr){
    if(arr == null)
        return null;
    int [] res = new int[arr.length];
    for(int i = 0 ; i < arr.length ; i++)
        res[i] = arr[i];
    return res;
}

//5.主函数
//可定义size，value  比对的次数times等等
public static void main(String[] args){
    int testtimes = 500000;
    int size = 10;
    int value = 100;
    boolean succeed = true;
    for(int i = 0 ; i < testtimes ; i++){
        int[] arr1 = generateRandomArray(size,value);
        int[] arr2 = copyArray(arr1);
        int[] arr3 = copyArray(arr1);
        insertSort(arr2);
        rightMathod(arr3);
        
        if(!isEqual(arr2,arr3)){
            succeed = false;
            printArray(arr1);  //打印出导致失败的用例   此方法过于easy，省略
            break;
        }
    }
    
    System.out.println(succeed ? "Nice!" : "Failed!");
}

```



## 写在最后

本文作为寒假学习总结的第一篇 Blog ，仅仅写了几个很基础的算法。

下一篇 Blog 内容灰常干货：递归分析，归并排序以及几个相关算法题。

