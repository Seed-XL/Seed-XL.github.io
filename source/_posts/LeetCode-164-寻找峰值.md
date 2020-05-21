---
title: LeetCode162 寻找峰值
date: 2020-05-17 17:24:17
categories : Algorithm
tags:
- LeetCode
- Algorithm
---

[题目原链接](https://leetcode-cn.com/problems/find-peak-element/)

题目大意就是给出一个数组，然后寻找其中的一个峰值的下标索引。这里有一个值得注意的地方是，数组边界nums[0]和nums[n-1]也是可能成为峰值的，因为默认nums[-1]和nums[n]是无穷小。

## 解题思路

### 寻找线索

先来分析一下题目，看看有没有什么线索可用。峰值，顾名思义，就是其值大于左右两侧的值，那么判断一个值，**最少需要3个数(暂不考虑特例)**。

### 确定算法

由于题目给出的说明：解法的时间复杂度应该是**O(logN)**，结合前面的性质，我想到的是：**分治法**。

顺着思路走，下一步就是该如何分解问题，来到这道题，对应的就是**如何划分数组**，为了简单起见，我每次将数组进行1/2划分。那么结合递归的写法，需要确定算法的**递归条件**和**基线条件**。

根据前面提到的数组切分，可以这样做，**当子数组的长度大于3的时候，就将数组进行1/2划分**，缩小问题的规模，这个是数组的递归条件。

根据前面提到的性质 ，算法的基线条件就是**当子数组的长度小于或等于3**的时候。小于3直接就返回，这种情况不存在峰值。等于3，尝试在其中的子数组找出峰值，并返回结果。

### 特殊情况 

但是这里有一个特例的情况，请看下面这个case:

```c#
[1,3,2,1]
```

如果单纯是按照在子数组中寻找峰值的话，最终算法会找不到峰值。因为它并没有考虑，**组成峰值的3个数有可能是跨越两个子数组**，因此需要处理左子数组和右子数组都合并起来的情况。根据峰值的性质，就算两个子数组合并，峰值也只会出现在两个子数组合并的边界，因此只需要简单地比较一下合并边界的值，即可判断是否存在峰值。



## 代码实现

```c#
public class Solution {
    public int FindPeakElement(int[] nums)
    {
        //特例，就是峰值是第一个元素和最后一个元素的情况
        if(  nums.Length <= 1 || nums[0] > nums[1]  )
        {
            return 0 ; 
        }
        else if( nums[nums.Length-1] > nums[nums.Length-2] )
        {
            return nums.Length - 1 ; 
        }
        else
        {
            return FindPeekValue(nums,0,(nums.Length + 0) >> 1, nums.Length-1)  ; 
        }
    }

    private int FindPeekValue( int[] nums,int s ,int m, int e)
    {
        int len = e + 1 - s   ;
        if( len > 3 )
        {
            //递归条件
            
            //左半子数组寻找
            int ret = FindPeekValue(nums,s,(m-1+s) >> 1,m-1) ;
            if( ret != -1 )
            {
                return ret ; 
            }
            //右半子数组寻找
            ret = FindPeekValue(nums,m,(e + m) >> 1,e)  ; 
            if( ret != -1 )
            {
                return ret; 
            }
            
            //合并两段来找
            //峰值既可能在左子数组，又有可能在右子数组的情况 ，需要区分。
            return ( nums[m] > nums[m-1] && nums[m] > nums[m+1] ) ? m 
            : ( nums[m-1] > nums[m-2] && nums[m-1] > nums[m] ? m-1 : -1 )  ; 
        }
        else if( len == 3 )
        {
            //基线条件 len == 3
            return ( nums[m] > nums[m-1] && nums[m] > nums[m+1] ) ? m : -1  ;
        }

        //基线条件 len < 3
        return -1 ;
    }

}
```

