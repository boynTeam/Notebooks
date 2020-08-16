---
title: 面试题30 包含min函数的栈
date: 2019-10-27 10:19:57
tags:
	- 算法
	- 刷题
	- 剑指Offer
	- 栈
categories:
	- 算法
	- 刷题
---

## 问题描述

 定义栈的数据结构，请在该类型中实现一个能够得到栈中所含最小元素的min函数（时间复杂度应为O（1））。 

<!--more-->

## 思路

创建一个辅助栈,里面记录最小值,与数据栈一起组成两个栈.行为是这样的:当数据栈为空,直接将数据入栈,同时也将这个值入辅助栈.注意一点,辅助栈的size和数据栈的size是一致的.当数据栈不为空,将数据入栈,然后跟辅助栈的顶部比较,比顶部大,就将顶部值入辅助栈,比顶部小,将数据入辅助栈.

出栈时,将数据栈和辅助栈双双出栈.调用min函数就返回辅助栈的栈顶.

总结起来思路就是辅助栈指示数据栈中每一个元素对应的总体最小值.

## 编码

```java
import java.util.Stack;

public class Solution {
    private Stack<Integer> data = new Stack<>();
    private Stack<Integer> min = new Stack<>();
    
    public void push(int node) {
        data.push(node);
        if(min.size()==0){
            min.push(node);
        }else{
            int mV = min.peek();
            if(node<mV) min.push(node);
            else min.push(mV);
        }
    }
    
    public void pop() {
        min.pop();
        data.pop();
    }
    
    public int top() {
        return data.peek();
    }
    
    public int min() {
        assert min.size() > 0;
        return min.peek();
    }
}
```

## 做题记录

| 状态 | 日期           | 评价               |
| ---- | -------------- | ------------------ |
| 👌    | 2019年10月27日 | 考察对栈的熟悉程度 |