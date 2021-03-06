## [[POJ](http://poj.org/)] [[INDEX](https://github.com/lyy289065406/POJ-Solving-Reports)] [1837] [[Balance](http://poj.org/problem?id=1837)]

> [Time: 1000MS] [Memory: 30000K] [难度: 初级] [分类: 背包]

------

## 问题描述

有一个天平，天平左右两边各有若干个钩子，总共有C个钩子，有G个钩码，求将钩码全部挂到钩子上使天平平衡的方法的总数。

其中可以把天枰看做一个以x轴0点作为平衡点的横轴

输入：

```
2 4 //C 钩子数 与 G钩码数
-2 3 //负数：左边的钩子距离天平中央的距离；正数：右边的钩子距离天平中央的距离c[k]
3 4 5 8 //G个重物的质量w[i]
```


## 解题思路

**提示：动态规划，01背包**

初看此题第一个冲动就是穷举。。。。不过再细想肯定行不通, O(20^20)等着超时吧。。。

我也是看了前辈的意见才联想到01背包，用动态规划来解。


**dp思路**：

每向天平中方一个重物，天平的状态就会改变，而这个状态可以由若干前一状态获得。


首先定义一个 **平衡度 j 的概念**：

**当平衡度j=0时，说明天枰达到平衡，j>0，说明天枰倾向右边（x轴右半轴），j<0则相反**

那么此时可以把平衡度j看做为衡量当前天枰状态的一个值，

因此可以定义一个 状态数组 `dp[i][j]`，意为在挂满前i个钩码时，平衡度为j的挂法的数量。

由于距离 `c[i]` 的范围是 `-15~15`，钩码重量的范围是 `1~25`，钩码数量最大是20

因此最极端的平衡度是所有物体都挂在最远端，**因此平衡度最大值为** `j=15*20*25=7500`。原则上就应该有 `dp[ 1~20 ][-7500 ~ 7500 ]`。

为了不让下标出现负数，做一个处理，使使得数组开为 `dp[1~20][0~15000]`，则当 `j=7500` 时天枰为平衡状态。

 

那么每次挂上一个钩码后，对平衡状态的影响因素就是每个钩码的 力臂：

　`力臂 = 重量 *臂长 = w[i] * c[k]`

那么若在挂上第i个砝码之前，天枰的平衡度为j

　(换言之把前 `i-1` 个钩码全部挂上天枰后，天枰的平衡度为 j )

则挂上第i个钩码后，即把前i个钩码全部挂上天枰后，天枰的平衡度 `j = j + w[i] * c[k]`

　其中 `c[k]` 为天枰上钩子的位置，代表第i个钩码挂在不同位置会产生不同的平衡度

 

不难想到，假设 `dp[i-1][j]` 的值已知，设 `dp[i-1][j] = num`

　　（即已知把前i-1个钩码全部挂上天枰后得到状态j的方法有num次）

　那么 `dp[i][ j+ w[i]*c[k] ] = dp[i-1][j] = num`

(即以此为前提，在第k个钩子挂上第i个钩码后，得到状态 `j + w[i] * c[k]` 的方法也为num次)

 

想到这里，利用**递归思想**，不难得出 **状态方程** `dp[i][ j + w[i] * c[k] ]= ∑(dp[i-1][j])`

有些前辈推导方式稍微有点不同，得到的 **状态方程为** `dp[i][j] = ∑(dp[i-1][j - c[i] * w[i]])`
 

其实**两条方程是等价的**，这个可以简单验证出来，而且若首先推导到第二条方程，也必须转化为第一条方程，这是为了避免下标出现负数。

------

**结论**： 最终转化为01背包问题

- 状态方程： `dp[i][ j + w[i] * c[k] ]= ∑(dp[i-1][j])`
- 初始化： `dp[0][7500] = 1`   不挂任何重物时天枰平衡，此为一个方法
- 复杂度： `O(C*G*15000)`  完全可以接受


## AC 源码


```c
//Memory Time 
//1496K  0MS 

//我所使用的解题方法，由于dp状态方程组申请空间比较大大
//若dp为局部数组，则会部分机器执行程序时可能由于内存不足会无法响应
//所以推荐定义dp为全局数组，优先分配内存

#include<iostream>
using namespace std;

int dp[21][15001]; //状态数组dp[i][j]
	                   //放入（挂上）前i个物品（钩码）后，达到j状态的方法数
int main(int i,int j,int k)
{
	int n;  //挂钩数
	int g;  //钩码数
	int c[21];  //挂钩位置
	int w[21];  //钩码重量

	
	/*Input*/

	cin>>n>>g;

	for(i=1;i<=n;i++)
		cin>>c[i];
	for(i=1;i<=g;i++)
		cin>>w[i];

	/*Initial*/

	memset(dp,0,sizeof(dp));  //达到每个状态的方法数初始化为0
	dp[0][7500]=1;     //7500为天枰达到平衡状态时的平衡度
	                   //放入前0个物品后，天枰达到平衡状态7500的方法有1个，就是不挂钩码

	/*DP*/

	for(i=1;i<=g;i++)
		for(j=0;j<=15000;j++)
			if(dp[i-1][j])  //优化，当放入i-1个物品时状态j已经出现且被统计过方法数，则直接使用统计结果
				            //否则忽略当前状态j
				for(k=1;k<=n;k++)
					dp[i][ j+w[i]*c[k] ] += dp[i-1][j]; //状态方程
	
	/*Output*/

	cout<<dp[g][7500]<<endl;
	return 0;
}
```

------

## 版权声明

　[![Copyright (C) EXP,2016](https://img.shields.io/badge/Copyright%20(C)-EXP%202016-blue.svg)](http://exp-blog.com)　[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
  

- Site: [http://exp-blog.com](http://exp-blog.com) 
- Mail: <a href="mailto:289065406@qq.com?subject=[EXP's Github]%20Your%20Question%20（请写下您的疑问）&amp;body=What%20can%20I%20help%20you?%20（需要我提供什么帮助吗？）">289065406@qq.com</a>


------
