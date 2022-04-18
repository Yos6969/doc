# 动态规划

![image-20220414021105832](.\img\image-20220414021105832.png)



## 01背包

01背包问题是最基本的背包问题，其题意可大概描述为一共有N件物品，每件物品都有其相应的体积和价值，给你一个背包，背包有容量上限，怎样往背包中装物品，能让背包中的物品价值最高。

问题：

A:每件物品消耗的容量  A = [2,3,4,5]

V:每件物品的价值 V = [3,4,5,6]

此类问题是典型的动态规划问题，动态规划的目标是背包中物品的总价值，动态规划的变量是物品的体积和价值，以及背包的容量上限，经过分析，我们可以定义一下dp状态：
       dp/[ i ]/[ j ] ——在背包容量为j时，从前i件物品里挑选装入背包可得到的最大价值 
       dp/[ i ]/[ j ]有两种情况：1. 第i件物品不装入背包 2.第i件物品装入背包（装入背包需要满足物品装入背包后背包容量没有超限）
       据此我们可以得到状态转移方程：

```c++
dp[i][j] = max(dp[i-1][j], dp[i-1][j-A[i]]+V[i]); if(j-A[i]>=0)

vector<int> A={2,3,4,5};
vector<int> V={3,4,5,6};    
//背包容量为10
vector<vector<int>> dp(V.size()+1, vector<int>(11));
    for (int i =1 ; i<=V.size(); i++)
    {
        for (int j =0 ; j <dp[0].size(); j++)
        {
            dp[i][j] = dp[i-1][j];
            if(j - A[i-1] >= 0)
            {
                dp[i][j] = max(dp[i-1][j], dp[i-1][j - A[i-1]] + V[i-1]);
            }
        }
    }
//优化空间，只能倒序，否则一个物品相当于添加了多次---完全背包可以正序
    vector<int> dp(11);
    for (int i = 1 ; i <= V.size(); i++)
    {
        for(int j = dp.size()-1; j>=0 ; j--)
        {
            if(j-A[i-1] >= 0)
            {
                dp[j] = max(dp[j],dp[j-A[i-1] ] + V[i-1] );
            }

        }
    }
```

## 完全背包

枚举每件物品取 0，1，2，3...m/A[I] 件

```
dp[i][j] = max(dp[i-1][j],dp[i-1][j-k*A[i]] + k*V[i])
```

可以转换为 0 1 背包，只要把完全背包多出来的件数等价于一件新的物品就行

```c++
if    vector<int> V={30,50,100,200};
    vector<int> A={2,3,4,5};


    vector<vector<int>> dp(V.size()+1, vector<int>(9));

    for (int i =1 ; i<dp.size(); i++)
    {
        for (int j =0 ; j <dp[0].size(); j++)
        {
            for(int k =0 ; k *A[i-1] <= j;k++) { 
                if (j - k*A[i-1] >= 0) {
                    dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - k*A[i-1]] + k*V[i-1]);
                }
            }
        }
    }
//优化空间
	vector<int> dp(9);
    for (int i =1 ; i<=V.size(); i++)
    {
        for (int j =0 ; j <dp.size(); j++)
        {
            for(int k = 0 ; k *A[i-1] <= j;k++) { 
                if (j - k*A[i-1] >= 0) {
                    dp[j] = max(dp[j], dp[j - k*A[i-1]] + k*V[i-1]);
                }
            }
        }
    }
```



