Sunday 算法是 Daniel M.Sunday 于 1990 年提出的字符串模式匹配。其效率在匹配随机的字符串时比其他匹配算法还要更快。Sunday 算法的实现可比 KMP，BM 的实现容易太多。

## 算法过程

假定主串为 "HERE IS A SIMPLE EXAMPLE"，模式串为 "EXAMPLE"。

（1）

![](https://resource.ethsonliu.com/image/20190815_01.png)

从头部开始比较，发现不匹配。则 Sunday 算法要求如下：找到主串中位于模式串后面的第一个字符，即红色箭头所指的 "空格"，再在模式串中从后往前找 "空格"，没有找到，则直接把模式串移到 "空格" 的后面。

（2）

![](https://resource.ethsonliu.com/image/20190815_02.png)

依旧从头部开始比较，发现不匹配。找到主串中位于模式串后面的第一个字符 L，模式串中存在 L，则移动模式串使两个 L 对齐。


（3）

![](https://resource.ethsonliu.com/image/20190815_03.png)

找到匹配。

## 完整代码

```c++
#include <iostream>
#include <string>

#define MAX_CHAR 256
#define MAX_LENGTH 1000

using namespace std;

void GetNext(string & p, int & m, int next[])
{
	for (int i = 0; i < MAX_CHAR; i++)
		next[i] = -1;
	for (int i = 0; i < m; i++)
		next[p[i]] = i;
}

void Sunday(string & s, int & n, string & p, int & m)
{
	int next[MAX_CHAR];
	GetNext(p, m, next);

	int j;  // s 的下标
	int k;  // p 的下标
	int i = 0;
	bool is_find = false;
	while (i <= n - m)
	{
		j = i;
		k = 0;
		while (j < n && k < m && s[j] == p[k])
			j++, k++;

		if (k == m)
		{
			cout << "在主串下标 " << i << " 处找到匹配\n";
			is_find = true;
		}

		if (i + m < n)
			i += (m - next[s[i + m]]);
		else
			break;
	}

	if (!is_find)
		cout << "未找到匹配\n";
}

int main()
{
	string s, p;
	int n, m;

	while (cin >> s >> p)
	{
		n = s.size();
		m = p.size();
		Sunday(s, n, p, m);
		cout << endl;
	}

	return 0;
}
```

数据测试如下：

```plaintext
here#is#a#example
example
在主串下标 10 处找到匹配

aaa
a
在主串下标 0 处找到匹配
在主串下标 1 处找到匹配
在主串下标 2 处找到匹配

aaa
b
未找到匹配
```