---
title: LeetCode64 最小路径和
date: 2020-07-15 23:45:21
categories : Algorithm
tags:
- LeetCode
- Algorithm
---

### 题目

```
给定一个包含非负整数的 m x n 网格，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。

说明：每次只能向下或者向右移动一步。

示例:

输入:
[
  [1,3,1],
  [1,5,1],
  [4,2,1]
]
输出: 7
解释: 因为路径 1→3→1→1→1 的总和最小。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/minimum-path-sum
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
```

[题目原链接](https://leetcode-cn.com/problems/minimum-path-sum/)

题目大意就是给出一个非负整数m x n的网络，找出从左上角到右下角，路径和为最小的数。

### 暴力递归回溯

对于这类题目，第一反应就是暴力递归回溯法，这种方法遍历所有可行的情况，然后找出最少的和，但是稍微分析一下的话，就知道时间复杂度是O(2^(m+n))，肯定会超时，所以不予考虑，代码实现比较简单，这里也不贴出来了。

> 因为是从左上角到右下角，每次只能往下或者往右，那么无论怎么走，从左上角到右下角，都要进行
> (m - 1) + (n-1)次选择（将行和列拆分来看，按行走，需要走m-1次，按列走，需要走n-1次)。而又因为每次进行选择的时候，都只能往下或者往右走，那每次选择都2种不同走向，于是就有：O(2^(m+n))，这就是所有可能的情况的数量。



### 单源最短路径

然后最近刚好在看[《算法图解》](https://book.douban.com/subject/26979890/)的**狄克斯特拉算法**章节，刚好这道题抽象出来求的就是**单源最短路径**的问题，于是尝试用这个算法来解一下这道题，可惜在倒数第二个case超时了，看来时间复杂度还是过高了。

> 将这个m x n的网格，看成是带非负加权的有向图，每个元素指向其右侧和下侧的元素，元素的侧代表的是相应的加权。那么求从左上角到右下角的最小和路径，求的就是从左上角到右下角的最短路径。而用Dijkstra算法的复杂度是O(n^2)，在这里就是O(m*n)。



#### 代码实现

```c#
    /*
    [[1,3,1],[1,5,1],[4,2,1]]
    [[1,2],[2,1]]
    [[1,3,1,1],[1,5,1,6],[4,2,1,5]]
    [[1]]
    [[1,2,3,4]]
    [[1],[2],[3],[1]]
    [[0]]
    [[0],[1],[3]]
    [[0,0,0,0]]
    [[0],[0],[0],[0]]
    */
    
    
    public class Solution
    {
        private int[][] graph;
        private int rowMax = 0 ;
        private int colMax = 0 ;
        private BitArray processedNode;

        //index -> cost
        private Dictionary<int,int> costDict;
        //index -> index
        private Dictionary<int,int> parentDict;

        public int MinPathSum(int[][] grid)
        {
            //todo boundary case  

            graph = grid;  

            rowMax = grid.Length;
            if( rowMax > 0  )
            {
                colMax = grid[0].Length;
            }

            if( rowMax * colMax == 0 )
            {
                return 0 ; 
            }
            
            processedNode = new BitArray(rowMax * colMax);
            costDict = new Dictionary<int, int>(5);
            parentDict = new Dictionary<int,int>(5);

            List<KeyValuePair<int, int>> neighbors = new List<KeyValuePair<int, int>>()
            {
                new KeyValuePair<int, int>(1, 0),
                new KeyValuePair<int, int>(0, 1)
            };


            //dir
            KeyValuePair<int, int> downDir = new KeyValuePair<int, int>(1, 0);
            KeyValuePair<int, int> rightDir = new KeyValuePair<int, int>(0, 1);

            //初始
            for( int i = 0; i < neighbors.Count;++i)
            {
                var neighborDir = neighbors[i]; 
                int cost = GetNeighborCost(0, 0, ref neighborDir);
                if (cost >= 0)
                {
                    int neighborIdx = GetIndex(neighborDir.Key, neighborDir.Value);
                    SetOrAddCost(neighborIdx ,cost) ;
                    SetParent(GetIndex(0, 0),neighborIdx);
                }
            }
            //SetOrAddCost(GetIndex(rowMax - 1, colMax - 1), int.MaxValue); 


            int leastCost = -1;
            int leastCostNodeIdx = FindLostCostNode(out leastCost); 
            while ( leastCostNodeIdx >= 0 )
            {
                for( int i = 0 ; i < neighbors.Count; ++i)
                {
                    var neighborDir = neighbors[i];
                    int neighborCost = GetNeighborCost(leastCostNodeIdx,ref neighborDir);
                    if ( neighborCost >= 0 )
                    {
                        int neighborIdx = GetNeighborIdx(leastCostNodeIdx, ref neighborDir); 
                        int oldCost = -1; 
                        int newCost = leastCost + neighborCost;

                        bool isNewCost = !costDict.TryGetValue(neighborIdx, out oldCost);

                        if (  oldCost > newCost )
                        {
                            costDict[neighborIdx] = newCost;
                            parentDict[neighborIdx] = leastCostNodeIdx;
                        }
                        else if( isNewCost )
                        {
                            costDict.Add(neighborIdx, newCost);
                            parentDict[neighborIdx] = leastCostNodeIdx;  
                        }
                    }
                }

                processedNode.Set(leastCostNodeIdx, true);
                leastCostNodeIdx = FindLostCostNode(out leastCost);
            }

            int pathSum = graph[0][0] ; 
            int endCost = 0 ;
            if(costDict.TryGetValue(rowMax * colMax - 1,out endCost))
            {
                pathSum += endCost  ; 
            }
            return pathSum; 
        }

        private int GetIndex( int x ,int y )
        {
            return colMax * x + y; 
        }

        private bool ParseXY( int index, out int x ,out int y )
        {
            x = -1;
            y = -1; 

            if( index < 0 || index >= rowMax * colMax )
            {
                return false;
            }

            x = index / colMax;
            y = index % colMax;

            return true; 
        }

        private void SetOrAddCost(int index,int cost)
        {
            if (index < rowMax * colMax)
            {
                costDict[index] = cost;
            }
        }

        private void SetOrAddCost(int neighborX, int neighborY, int cost)
        {
            SetOrAddCost(GetIndex(neighborX, neighborY), cost);
        }

        private void SetParent( int parentNodeIdx ,int childNodeIdx )
        {
            int maxIdx = rowMax * colMax;
            if (parentNodeIdx < maxIdx && childNodeIdx < maxIdx)
            {
                parentDict[childNodeIdx] = parentNodeIdx; 
            }
        }

        private int GetNeighborIdx(int index, ref KeyValuePair<int, int> dir) 
        {
            int x = 0;
            int y = 0;
            if (ParseXY(index, out x, out y))
            {
                x += dir.Key;
                y += dir.Value;

                return GetIndex(x, y);
            }
            return -1;
        }
       

        private int GetNeighborCost( int index, ref KeyValuePair<int, int> dir )
        {
            int x = 0;
            int y = 0; 
            if( ParseXY(index,out x,out y) )
            {
                return GetNeighborCost(x, y, ref dir); 
            }

            return -1;
        }


        private int GetNeighborCost( int x , int y ,ref KeyValuePair<int, int> dir)
        {
            int neighborRow = x + dir.Key;
            int neighborCol = y + dir.Value;
            if (neighborRow < rowMax && neighborCol < colMax)
            {
                return graph[neighborRow][neighborCol];
            }
            return -1; 
        }

        private int FindLostCostNode( out int leastCost )
        {
            int index = -1; 
            leastCost = int.MaxValue ; 
            var etr = costDict.GetEnumerator(); 
            while (etr.MoveNext())
            {
                var nodeIdx = etr.Current.Key;
                var nodeCost = etr.Current.Value;

                if ( processedNode[nodeIdx] || nodeCost > leastCost )
                {
                    continue;
                }

                leastCost = nodeCost;
                index = nodeIdx;
            }

            return index ;
        }
    }
```



### 动态规划

鉴于用O(n^2)的算法也超时，那么基本就是要求用O(n)的算法来解了，结合题目求最值的走向，那么基本上就是要用动态规划来解了。恰好那会刚好在看[ labuladong的算法小抄](https://labuladong.github.io/ebook/)中关于动态规划的相关章节（写的非常通俗易懂），其中的内容提供了对应的思路，于是稍加分析之后就解出来了。



#### 递归解法

来看看题目，求左上角到右下角的最小路径和，那么是不是在求左下角向右或者向下走两条路径中的最小值呢？假如向右走是最小的，那么向右走了之后，再来走下一步，是不是一样在求向右或者向下走两条路径中的最小值呢？于是递归就产生了，不管我们怎么走，走的都是最小路径和的那一条路，于是可以得到：

> dp[x,y] =  {
> 						   Min( dp[x+1,y] , dp[x,y+1])   |   graph[x+1,y]  != null  &&  graph[x,y+1]  != null 
> 						   dp[x+1,y]							  |   graph[x+1,y]  != null  &&  graph[x,y+1]  == null
>                            dp[x,y+1]							  |   graph[x+1,y]  == null  && graph[x,y+1]  != null 
>                 }

![推导公式](https://raw.githubusercontent.com/Seed-XL/ArticlePicture/master/20200720003640.png)



##### 代码实现

```c#
    /*
        [[1,3,1],[1,5,1],[4,2,1]]
        [[1,2],[2,1]]
        [[1,3,1,1],[1,5,1,6],[4,2,1,5]]
        [[1]]
        [[1,2,3,4]]
        [[1],[2],[3],[1]]
        [[0]]
        [[0],[1],[3]]
        [[0,0,0,0]]
        [[0],[0],[0],[0]]
    */
    
    
    public class Solution
    {
        private int[][] graph;
        private int rowMax;
        private int colMax;

        private BitArray processedNode;
        private int[] gridTable; 
       

        public int MinPathSum(int[][] grid)
        {
            //todo boundary case  
            rowMax = grid.Length;
            if (rowMax > 0)
            {
                colMax = grid[0].Length;
            }

            if ( rowMax * colMax == 0 )
            {
                return 0;
            }

            processedNode = new BitArray(rowMax * colMax);
            gridTable = new int[rowMax * colMax];


            return MiniPathSum(grid, 0, 0); 
        }

        
        private int MiniPathSum( int[][] grid ,int x ,int y  )
        {
            int tableIdx = GetIndex(x, y);
            if ( tableIdx >= rowMax * colMax )
            {
                return -1 ; 
            }

            
            bool isCalced = processedNode[tableIdx]; 
            if( isCalced )
            {
                return gridTable[tableIdx];
            }

            //如果向下走，最小值是多少？
            int dirSumPath = MiniPathSum(grid, x + 1, y);
            //如果向右走，最小值又是多少呢？
            int rightSumPath = MiniPathSum(grid, x, y + 1);

            int miniDirSum = 0; 
            if( dirSumPath != -1 && rightSumPath != -1 )
            {
                //求向下向右走，两者间的最小值
                miniDirSum = Math.Min(dirSumPath, rightSumPath);
            }
            else if(dirSumPath != -1)
            {
                //向右走不了，那么最小值当然是向下走的路径的
                miniDirSum = dirSumPath; 
            }
            else if( rightSumPath != -1 )
            {
                //向下走不了，那么最小值当然是向右走的路径的。
                miniDirSum = rightSumPath; 
            }


            int pathSum = grid[x][y] + miniDirSum;  

            processedNode.Set(tableIdx, true);
            //DP表记录数值，后续直接查表减少重复计算
            gridTable[tableIdx] = pathSum;

            return pathSum; 
        }

        private int GetIndex(int x, int y)
        {
            return colMax * x + y;
        }

    }
```



#####  解题结果

![递归解法](https://raw.githubusercontent.com/Seed-XL/ArticlePicture/master/20200720003938.png)



#### 迭代解法

再来看看动态规划的递归解法 ，我们是从左上角的视角出发一步步递归下去，直到查到终点再返回的，属于自顶向下的解法 。那么能不能站在右下角的视角出发，反过来算呢？当然是可以的，因为换成终点视角来看的话，就变成了求“到当前的结点之前的最小路径和”，于是有：

> dp[x,y] =  {
> 						   Min( dp[x-1,y] , dp[x,y-1])   |   graph[x-1,y]  != null  &&  graph[x,y-1]  != null 
> 						   dp[x-1,y]							  |   graph[x-1,y]  != null  &&  graph[x,y-1]  == null
>                            dp[x,y-1]							  |   graph[x-1,y]  == null  && graph[x,y-1]  != null 
>                 }

![状态转移公式](https://raw.githubusercontent.com/Seed-XL/ArticlePicture/master/20200720011124.png)

##### 代码实现

```c#
    /*
        [[1,3,1],[1,5,1],[4,2,1]]
        [[1,2],[2,1]]
        [[1,3,1,1],[1,5,1,6],[4,2,1,5]]
        [[1]]
        [[1,2,3,4]]
        [[1],[2],[3],[1]]
        [[0]]
        [[0],[1],[3]]
        [[0,0,0,0]]
        [[0],[0],[0],[0]]
    */
    
    
    public class Solution
    {
        private int rowMax;
        private int colMax;

        private BitArray processedNode;
        private int[] gridTable; 
       

        public int MinPathSum(int[][] grid)
        {
            //todo boundary case  
            rowMax = grid.Length;
            if (rowMax > 0)
            {
                colMax = grid[0].Length;
            }

            if ( rowMax * colMax == 0 )
            {
                return 0;
            }

            processedNode = new BitArray(rowMax * colMax);
            gridTable = new int[rowMax * colMax];

            int maxTableIdx = rowMax * colMax ; 
            for( int i = 0 ; i < maxTableIdx ; ++i )
            {
                int x = -1 ; 
                int y = -1 ;
        
                if( ParseXY(i,out x,out y) )
                {
                    int dx = x - 1 ; 
                    int ry = y - 1 ; 
                    //先查一下别人向下走到我这里，也就是我向上的最小路径和
                    int downSum = GetDpTableSum(GetIndex(dx,y)) ; 
                    //再查一下别人向右走到我这里，也就是我向左的最小路径和
                    int rightSum = GetDpTableSum(GetIndex(x,ry)) ;

                    int miniDirSum = 0  ; 
                    
                    if( downSum != -1 && rightSum != -1 )
                    {
                        //两条路都可以走到我这里喔，选一个更加小的吧
                        miniDirSum = Math.Min(downSum, rightSum);
                    }
                    else if(downSum != -1)
                    {
                        //别人不能向右走到我这里，但是可以向下走到我这里，那么你是最小的
                        miniDirSum = downSum; 
                    }
                    else if( rightSum != -1 )
                    {
                        //别人不能向下走到我这里，但是可以向右走到我这里，那么你是最小的
                        miniDirSum = rightSum; 
                    }
				
                    //那么到我这个位置的最小值 ，就是 我当前的值 + 到我这里之前的最小值
                    gridTable[i] =  grid[x][y] + miniDirSum  ;    
                } 
            }


            return gridTable[maxTableIdx-1]  ; 
        }

        
        private int GetDpTableSum( int index )
        {
            if( index < 0 || index >= (colMax * rowMax) )
            {
                return -1 ;              
            }

            return gridTable[index] ; 
        }

        private int GetIndex(int x, int y)
        {
            return colMax * x + y;
        }

        private bool ParseXY( int index, out int x ,out int y )
        {
            x = -1;
            y = -1; 

            if( index < 0 || index >= rowMax * colMax )
            {
                return false;
            }

            x = index / colMax;
            y = index % colMax;

            return true; 
        }
        

    }
```



##### 解题结果

![迭代解法](https://raw.githubusercontent.com/Seed-XL/ArticlePicture/master/20200720004235.png)

