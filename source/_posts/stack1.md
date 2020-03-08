---
title: 栈操作（一）
date: 2020-03-08 23:12:45
tags: [Java,stack]
categories: 每日刷题
---



## 一、两个栈实现队列，支持队列基本操作

### 【思路】

栈的特点是 FILO ，队列的特点是 FIFO，用两个栈正好可以实现队列结构。

<!-- more -->

实现方法：一个栈 stackPush 负责接收压入的数据，另一个栈 stackPop 负责弹出数据。只需要在数据压入 stackPush 后，再导入 stackPop中即可。

![two stacks](/images/stack1.jpg)

<center>图1 两个栈的操作</center>

### 【两个原则】

要想成功的倒数据 ，必须遵循两个原则，否则就会发生意想不到的错误：

1. stackPush 一旦要向 stackPop 中倒数据，则必须完全倒干净，不能只倒一部分；
2. 如果 stackPop 不为空，那么一定不能往 stackPop 中倒数据。



违反1的情况举例：1 ~ 5依次压入stackPush，stackPush的栈顶到栈底为5 ~ 1，从stackPush压入stackPop时，只将5和4压入了stackPop，stackPush还剩下1、2、3没有压入。此时如果用户想进行弹出操作，那么4将最先弹出，与预想的队列顺序就不一致。

违反2的情况举例：1 ~ 5依次压入stackPush，stackPush将所有的数据压入stackPop，此时从stackPop的栈顶到栈底就变成了1 ~ 5。此时又有6 ~ 10依次压入stackPush，stackPop不为空，stackPush不能向其中压入数据。如果违反2压入了stackPop，从stackPop的栈顶到栈底就变成了6 ~ 10、1 ~ 5。那么此时如果用户想进行弹出操作，6将最先弹出，与预想的队列顺序就不一致



“倒数据”行为可以发生在 push、pop、peek的任何一个过程中。主要满足上面两个规则，就绝对不会出错。

下面的 code 中我将“倒数据”定义为 pushPop 。



### 【实现 Code】

```java
public class StackToQueue{
    public Stack<Integer> stackPush;
    public Stack<Integer> stackPop;
    
    public StackToQueue(){
        stackPush = new Stack<Integer>();
        stackPop = new Stack<Integer>();
    }
    
    //从stackPush中向stackPop倒数据
    private void pushPop(){
        if(stackPop().isEmpty()){
            while(!stackPush().isEmpty()){
                stackPop.push(stackPush.pop());
            }
        }
    }
    
    //往队尾加数据
    public void add(int pushInt){
        stackPush.push(pushInt);
        pushPop();
    }
    
    //从队头出数据
    public int poll(){
        if(stackPop.isEmpty() && stackPush.isEmpty()){
            throw new RuntimeException("Queue is EMPTY!");
        }
        pushPop();
        return stackPop.pop();
    }
    
    //看看队头数据
    public int peek(){
        if(stackPop.isEmpty() && stackPush.isEmpty()){
            throw new RuntimeException("Queue is EMPTY!");
        }
        pushPop();
        return stackPop.peek();
    }
 
}
```



----



## 二、仅用 递归函数 逆序一个栈

### 【题目】

一个栈依次压入1、2、3、4、5，那么从栈顶到栈底分别为5、4、3、2、1。将这个栈转置后，从栈顶到栈底为1、2、3、4、5，也就是实现栈中元素的逆序，但是只能用递归函数来实现，不能用其他数据结构。

解释一下哈，也许你会想到用两个栈，但这题仅能使用一个栈（否则岂不是太easy？？）



### 【思路】

就笔者个人经验来说，递归函数咱们不能想的太多。如果每一步操作都想看得透透的，其实我们初学者往往会囿于其中无法走出。咱们只需要想清楚：我们希望它实现什么；什么时候返回。往往把这两件事做好，我们写的函数八九不离十是正确的。



那这道题来说，我们要想逆序一个栈，首先要拿出这个数的栈底元素，准备将其放在最顶上；然后在压入时，原来的栈底元素必须最后一个压，但是由于我们最先拿到的是栈顶，所以应该在压的时候做一个逆序。如此，这道题就解了。



### 【实现 Code】

getLastElem：拿到栈底元素

```java
public int getLastElem(Stack<Integer> stack){
    int result = stack.pop();//此时拿到栈顶，but需要判断result是不是最下面的元素
    if(stack.isEmpty()){
        return result;//是，拿到结果了，返回
    }else{
        int last = getLastElem(stack);//继续获取下面的元素
        stack.push(result);//因为result不是最下面的，所以压回
        return last;
    }
}
```

假设栈顶到栈底依次是 3，2，1，这个函数的具体过程如下：

![getLastElem(stack)](/images/stack2.jpg)

<center>图2 getLastElem的调用过程 </center>

reverse：将整个栈逆序

```java
public void reverse(Stack<Integer> stack){
    if(stack.isEmpty()){
        return;
    }
    int elem = getLastElem(stack);//i已经是栈底元素了
    reverse(stack);//将整个栈逆序后，elem压入后才是栈顶
    stack.push(elem);
}
```



假设栈顶到栈底依次是 3，2，1，这个函数的具体过程如下：

![reverse(stack)](/images/stack3.jpg)

<center>图3 reverse的调用过程 </center>

----

