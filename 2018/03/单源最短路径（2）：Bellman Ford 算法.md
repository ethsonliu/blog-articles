Dijkstra 算法是处理单源最短路径的有效算法，但它对存在负权回路的图就会失效。这时候，就需要使用其他的算法来应对这个问题，Bellman-Ford（中文名：贝尔曼-福特）算法就是其中一个。

Bellman-Ford 算法不仅可以求出最短路径，也可以检测负权回路的问题。该算法由美国数学家理查德•贝尔曼（Richard Bellman, 动态规划的提出者）和小莱斯特•福特（Lester Ford）发明。

## 算法过程分析

对于一个不存在负权回路的图，Bellman-Ford 算法求解最短路径的方法如下：

设其顶点数为 n，边数为 m。设其源点为 source，数组 `dist[i]` 记录从源点 source 到顶点i的最短路径，除了 `dist[source]` 初始化为 0 外，其它 `dist[]` 皆初始化为 INT_MAX。以下操作循环执行 n-1 次：

*  对于每一条边 arc(u, v)，如果 dist[u] + w(u, v) < dist[v]，则使 dist[v] = dist[u] + w(u, v)，其中 w(u, v) 为边 arc(u, v) 的权值。

n-1 次循环，Bellman-Ford 算法就是利用已经找到的最短路径去更新其它点的 `dist[]`。

接下来再看看 Bellman-Ford 算法是如何检测负权回路的？

![](https://cdn.ethsonliu.com/x1/20180331_02.png)

检测的方法很简单，只需在求解最短路径的 n-1 次循环基础上，再进行第 n 次循环：

* 对于所有边，只要存在一条边 arc(u, v) 使得 dist[u] + w(u, v) < dist[v]，则该图存在负权回路，其中 w(u, v) 为边 arc(u, v) 的权值。

| 循环次数 | dist[0] | dist[1] | dist[2] |
| :--: | :-----: | :-----: | :-----: |
| 第1次  |    0    |   -5    |   -3    |
| 第2次  |   -2    |   -5    |   -3    |
| 第3次  |   -2    |   -7    |   -5    |

## 完整代码

```c++
#include <iostream>
#include <stack>

using namespace std;

struct Edge
{
	int u;
	int v;
	int w;
};

Edge edge[10000]; // 记录所有边
int  dist[100];   // 源点到顶点 i 的最短距离
int  path[100];   // 记录最短路的路径
int  vertex_num;  // 顶点数
int  edge_num;    // 边数
int  source;      // 源点

bool BellmanFord()
{
	// 初始化
	for (int i = 0; i < vertex_num; i++)
		dist[i] = (i == source) ? 0 : INT_MAX;

	// n-1 次循环求最短路径
	for (int i = 1; i <= vertex_num - 1; i++)
	{
		for (int j = 0; j < edge_num; j++)
		{
			if (dist[edge[j].v] > dist[edge[j].u] + edge[j].w)
			{
				dist[edge[j].v] = dist[edge[j].u] + edge[j].w;
				path[edge[j].v] = edge[j].u;
			}
		}
	}

	bool flag = true;  // 标记是否有负权回路

	// 第 n 次循环判断负权回路
	for (int i = 0; i < edge_num; i++)
	{
		if (dist[edge[i].v] > dist[edge[i].u] + edge[i].w)
		{
			flag = false;
			break;
		}
	}

	return flag;
}

void Print()
{
	for (int i = 0; i < vertex_num; i++)
	{
		if (i != source)
		{
			cout << source << " 到 " << i << " 的最短距离是：" << dist[i] << "，路径是：" << i;
			int t = path[i];
			while (t != source)
			{
				cout << "--" << t;
				t = path[t];
			}
			cout << "--" << source << endl;
		}
	}
}

int main()
{

	cout << "请输入图的顶点数，边数，源点：";
	cin >> vertex_num >> edge_num >> source;

	cout << "请输入 " << edge_num << " 条边的信息：\n";
	for (int i = 0; i < edge_num; i++)
		cin >> edge[i].u >> edge[i].v >> edge[i].w;

	if (BellmanFord())
		Print();
	else
		cout << "存在负权回路!\n";

	return 0;
}
```

测试如下：

```plaintext
/* Test 1 */
请输入图的顶点数，边数，源点：5 7 0
请输入 7 条边的信息：
0 1 100
0 2 30
0 4 10
2 1 60
2 3 60
3 1 10
4 3 50
0 到 1 的最短距离是：70，路径是：1--3--4--0
0 到 2 的最短距离是：30，路径是：2--0
0 到 3 的最短距离是：60，路径是：3--4--0
0 到 4 的最短距离是：10，路径是：4--0

/* Test 2 */
请输入图的顶点数，边数，源点：4 6 0
请输入 6 条边的信息：
0 1 20
0 2 5
3 0 -200
1 3 4
3 1 4
2 3 2
存在负权回路!
```
## 算法优化

以下除非特殊说明，否则都默认是不存在负权回路的。

先来看看 Bellman-Ford 算法为何需要循环 n-1 次来求解最短路径？

读者可以从 Dijkstra 算法来考虑，想一下，Dijkstra 从源点开始，更新 `dist[]`，找到最小值，再更新 `dist[]` ，，，每次循环都可以确定至少一个点的最短路。Bellman-Ford 算法同样也是这样，它的每次循环也可以确定至少一个点的最短路，故需要 n-1 次循环。

Bellman-Ford 算法的时间复杂度为 $O(nm)$，其中 n 为顶点数，m 为边数。每一次循环都需要对 m 条边进行操作，$O(nm)$ 的时间，其实大多数都浪费了。

大家可以考虑一个随机图（点和边随机生成），除了已确定最短路的顶点与尚未确定最短路的顶点之间的边，其它的边所做的都是无用的，大致描述为下图（分割线以左为已确定最短路的顶点），

![](https://cdn.ethsonliu.com/x1/20180331_03.png)

其中红色部分为所做无用的边，蓝色部分为实际有用的边。

既然只需用到中间蓝色部分的边，那算法优化的方向就找到了，请接着看本系列第三篇文章：spfa 算法。
