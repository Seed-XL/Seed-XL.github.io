---
title: LeetCode179 最大数
date: 2020-07-12 23:35:30
categories: Algorithm
tags:
- LeetCode
- Algorithm
---

[题目原链接](https://leetcode-cn.com/problems/largest-number/)

题目大意是给出一个数组，里面包含一系列数字，你的任务是将这些数字组合成一个最大数，然后这个数字可能会很大，需要以字符串的形式给出。

---



## 解题思路

基本方向就是**按一定的顺序将数组中的数字进行排序**，然后依次按顺序取出数字组合成一个最大数，而这个**一定的顺序**，就是这道题的关键。
先来看一下一般情况 ：最高位数值不一次的数字：

```c#
1234
112
234
223
34
4 
```

显而易见，对于这类数字，我们只需要**按其当前最高位的大小(如果最高位大小相同，则按次高位的大小，依次类推)**来排序就行。

那么，如果最高位或者接下来几位都相同呢？

```c#
824
8248
8246
8248241
8248249
```

是不是没那么容易看出来了？对于这些情况，最简单直接粗暴的做法，就是每个数字都彼此组合一遍，然后再比较一下谁大谁小，依此来排序，但是效率有点低。

让我们先来看看两组数字，找找规律，看看有没什么特殊性质利用一下。

```
case 1				case 2
 824				 824
 8246				 8248
   |      		         | 
 8248246			 8248248
 8246824			 8248824
```

case1和case2组成的最大数分别是**8248246**和**8248824**，两组数字都有点类似，就是最高位乃至后面几位都和另外一组数字相同。按照一般case的对比规则，对于case1，我们最终比较的结果就是， 824和8246在比较完数字**4**之后，数字**6**无数可比，那么**6**应该和谁比呢？

我们来这样想一下，如果我觉得**824**应该要排在**8246**前面，是不是以**824**开头的最大数**824X**要比以**8246**开头的最大数**8246**要大呢？反过来如果是**8246**要排在前面，那么一定是**8246**比**824X**大。那么如何决定这个X是什么呢？

答案就是**滑动窗口**。当**824**和**8246**比较完数字**4**之后，应该将**8246**中和**824**中相同的部分滑动去掉，将**6**当作新数字中的最高位：

```c#
---824
---8246
||
---824
8246
```

为什么是滑动窗口呢？因为当**824**遍历完之后，这个时候就需要进行**谁是第四个及后续数字**的判断。这需要拿数字比较多(**8246**)的那一组数字中剩余还没有比较的数字(数字**6**)，去和前面已经比较过的数字(**824**)进行比较，毕竟对于新的最大数开头那部分**824X**，X的值有可能是**6**(8246排前面)，或者是**8**(824排前面)。

那么根据前面的规律，将相同部分的数字滑动掉之后，又可以继续用normal case的规则去进行比较了，直到找到谁大谁小即可。

---



##  代码实现

```c#
	public class Solution
    {
        public string LargestNumber(int[] nums)
        {
            Array.Sort(nums, Sort);

            if (nums[0] == 0)
            {
                return "0";
            }

            string numStr = string.Empty;
            for (int i = 0; i < nums.Length; ++i)
            {
                List<int> numList = GetNumsSet(nums[i]);
                for (int j = numList.Count - 1; j >= 0; --j)
                {
                    numStr = string.Format("{0}{1}", numStr, numList[j]);
                }
            }

            return numStr;
        }

        private List<int> GetNumsSet(int num)
        {
            List<int> lNumsList = new List<int>(5);
            if (num >= 10)
            {
                int lq = 0;
                int lr = 0;
                do
                {
                    lq = num / 10;
                    lr = num % 10;

                    lNumsList.Add(lr);
                    num = lq;

                } while (lq > 0);
            }
            else
            {
                lNumsList.Add(num);
            }

            return lNumsList;
        }

        private int CompareRecusive(List<int> lList, List<int> rList)
        {
            int iResult = 0;
            int lIdx = lList.Count - 1;
            int rIdx = rList.Count - 1;

            // normal case 从最高位依次比较下去
            while (lIdx >= 0 && rIdx >= 0)
            {
                int lValue = lList[lIdx];
                int rValue = rList[rIdx];

                if (lValue == rValue)
                {
                    iResult = 0;
                }
                else
                {
                    iResult = lValue > rValue ? -1 : 1;
                    break;
                }

                lIdx--;
                rIdx--;
            }

            if ( iResult == 0 )
            {
                //右边的数字多一些
                if (lList.Count < rList.Count && rIdx >= 0 )
                {
                    //将右边的数字滑动，对齐左边的数字
                    var newRList = new List<int>();
                    for (int i = rIdx; i >= 0 ; --i)
                    {
                        newRList.Insert(0,rList[i]);
                    }

                    //滑动之后就可以采用normal case的比较方法了
                    iResult = CompareRecusive(lList, newRList);
                }
                else if( lIdx >= 0 )
                {
                    //左边的数字多一些，滑动左边的数字，以对齐右边的数字
                    var newLList = new List<int>();
                    for (int i = lIdx; i >=  0 ; --i)
                    {
                        newLList.Insert(0,lList[i]);
                    }
                    
                    //滑动之后就可以采用normal case的比较方法了
                    iResult = CompareRecusive(newLList, rList);
                }
            }

            return iResult;
        }

        private int Sort(int l, int r)
        {
            if (l == r)
            {
                return 0;
            }
            else
            {
                List<int> lList = GetNumsSet(l);
                List<int> rList = GetNumsSet(r);

                return CompareRecusive(lList, rList);
            }
        }

    }
```





## 解题结果

![result](https://raw.githubusercontent.com/Seed-XL/ArticlePicture/master/20200713003552.png)



