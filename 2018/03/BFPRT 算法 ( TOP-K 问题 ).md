## 一：背景介绍
在一堆数中求其前 k 大或前 k 小的问题，简称 TOP-K 问题。而目前解决 TOP-K 问题最有效的算法即是 **BFPRT 算法**，又称为**中位数的中位数算法**，该算法由 Blum、Floyd、Pratt、Rivest、Tarjan 提出，最坏时间复杂度为 $O(n)$。

在首次接触 TOP-K 问题时，我们的第一反应就是可以先对所有数据进行一次排序，然后取其前 k 即可，但是这么做有两个问题：

1. 快速排序的平均复杂度为 $O(nlogn)$，但最坏时间复杂度为 $O(n^2)$，不能始终保证较好的复杂度；
2. 我们只需要前 k 大的，而对其余不需要的数也进行了排序，浪费了大量排序时间。

除这种方法之外，堆排序也是一个比较好的选择，可以维护一个大小为 k 的堆，时间复杂度为 $O(nlogk)$。

那是否还存在更有效的方法呢？我们来看下 BFPRT 算法的做法。

**在快速排序的基础上**，首先通过判断主元位置与k的大小使递归的规模变小，其次通过修改快速排序中**主元的选取方法**来降低快速排序在**最坏情况下的时间复杂度**。

下面先来简单回顾下快速排序的过程，以升序为例：

1. 选取主元；
2. 以选取的主元为分界点，把小于主元的放在左边，大于主元的放在右边；
3. 分别对左边和右边进行递归，重复上述过程。

## 二：算法过程及代码

BFPRT 算法步骤如下：

1. 选取主元；
  1.1. 将 n 个元素按顺序分为 $⌊\frac n5⌋$ 个组，每组 5 个元素，若有剩余，舍去；
  1.2. 对于这 $⌊\frac n5⌋$ 个组中的每一组使用插入排序找到它们各自的中位数；
  1.3. 对于 1.2 中找到的所有中位数，调用 BFPRT 算法求出它们的中位数，作为主元；
2. 以 1.3 选取的主元为分界点，把小于主元的放在左边，大于主元的放在右边；
3. 判断主元的位置与 k 的大小，有选择的对左边或右边递归。

上面的描述可能并不易理解，先看下面这幅图：

![](https://resource.ethsonliu.com/image/20180325_01.png)

BFPRT() 调用 GetPivotIndex() 和 Partition() 来求解第 k 小，在这过程中，GetPivotIndex() 也调用了 BFPRT()，即 GetPivotIndex() 和 BFPRT() 为互递归的关系。

下面为代码实现，其所求为**前 k 小的数**：

```c++
#include <iostream>
#include <algorithm>

using namespace std;

int InsertSort(int array[], int left, int right);
int GetPivotIndex(int array[], int left, int right);
int Partition(int array[], int left, int right, int pivot_index);
int BFPRT(int array[], int left, int right, int k);

int main()
{
	int k = 8; // 1 <= k <= array.size
	int array[20] = { 11,9,10,1,13,8,15,0,16,2,17,5,14,3,6,18,12,7,19,4 };

	cout << "原数组：";
	for (int i = 0; i < 20; i++)
		cout << array[i] << " ";
	cout << endl;

	// 因为是以 k 为划分，所以还可以求出第 k 小值
	cout << "第 " << k << " 小值为：" << array[BFPRT(array, 0, 19, k)] << endl;

	cout << "变换后的数组：";
	for (int i = 0; i < 20; i++)
		cout << array[i] << " ";
	cout << endl;

	return 0;
}

/**
 * 对数组 array[left, right] 进行插入排序，并返回 [left, right]
 * 的中位数。
 */
int InsertSort(int array[], int left, int right)
{
	int temp;
	int j;

	for (int i = left + 1; i <= right; i++)
	{
		temp = array[i];
		j = i - 1;

		while (j >= left && array[j] > temp)
        {
            array[j + 1] = array[j];
            j--;
        }

		array[j + 1] = temp;
	}

	return ((right - left) >> 1) + left;
}

/**
 * 数组 array[left, right] 每五个元素作为一组，并计算每组的中位数，
 * 最后返回这些中位数的中位数下标（即主元下标）。
 *
 * @attention 末尾返回语句最后一个参数多加一个 1 的作用其实就是向上取整的意思，
 * 这样可以始终保持 k 大于 0。
 */
int GetPivotIndex(int array[], int left, int right)
{
	if (right - left < 5)
		return InsertSort(array, left, right);

	int sub_right = left - 1;

	// 每五个作为一组，求出中位数，并把这些中位数全部依次移动到数组左边
	for (int i = left; i + 4 <= right; i += 5)
	{
		int index = InsertSort(array, i, i + 4);
		swap(array[++sub_right], array[index]);
	}

	// 利用 BFPRT 得到这些中位数的中位数下标（即主元下标）
	return BFPRT(array, left, sub_right, ((sub_right - left + 1) >> 1) + 1);
}

/**
 * 利用主元下标 pivot_index 进行对数组 array[left, right] 划分，并返回
 * 划分后的分界线下标。
 */
int Partition(int array[], int left, int right, int pivot_index)
{
	swap(array[pivot_index], array[right]); // 把主元放置于末尾

	int partition_index = left; // 跟踪划分的分界线
	for (int i = left; i < right; i++)
	{
		if (array[i] < array[right])
		{
			swap(array[partition_index++], array[i]); // 比主元小的都放在左侧
		}
	}

	swap(array[partition_index], array[right]); // 最后把主元换回来

	return partition_index;
}

/**
 * 返回数组 array[left, right] 的第 k 小数的下标
 */
int BFPRT(int array[], int left, int right, int k)
{
	int pivot_index = GetPivotIndex(array, left, right); // 得到中位数的中位数下标（即主元下标）
	int partition_index = Partition(array, left, right, pivot_index); // 进行划分，返回划分边界
	int num = partition_index - left + 1;

	if (num == k)
		return partition_index;
	else if (num > k)
		return BFPRT(array, left, partition_index - 1, k);
	else
		return BFPRT(array, partition_index + 1, right, k - num);
}
```

运行如下：

```plaintext
原数组：11 9 10 1 13 8 15 0 16 2 17 5 14 3 6 18 12 7 19 4
第 8 小值为：7
变换后的数组：4 0 1 3 2 5 6 7 8 9 10 12 13 14 17 15 16 11 18 19
```

##三：时间复杂度分析

BFPRT 算法在最坏情况下的时间复杂度是 $O(n)$，下面予以证明。令 $T(n)$ 为所求的时间复杂度，则有：

$$
T(n)≤T(\frac n 5)+T(\frac {7n}{10})+c⋅n\tag{c 为一个正常数}
$$

其中：

- $T(\frac n 5)$ 来自 GetPivotIndex()，n 个元素，5 个一组，共有 $⌊\frac n5⌋$ 个中位数；
- $T(\frac {7n}{10})$ 来自 BFPRT()，在 $⌊\frac n5⌋$ 个中位数中，主元 x 大于其中 $\frac 12⋅\frac n5=\frac n{10}$ 的中位数，而每个中位数在其本来的 5 个数的小组中又大于或等于其中的 3 个数，所以主元 x 至少大于所有数中的 $\frac n{10}⋅3=\frac {3n}{10}$ 个。即划分之后，任意一边的长度至少为 $\frac 3{10}$，在最坏情况下，每次选择都选到了 $\frac 7{10}$ 的那一部分。
- $c⋅n$ 来自其它操作，比如 InsertSort()，以及 GetPivotIndex() 和 Partition() 里所需的一些额外操作。

设 $T(n)=t⋅n$，其中 t 为未知，它可以是一个正常数，也可以是一个关于 n 的函数，代入上式：

$$
\begin{align}
t⋅n&≤\frac {t⋅n}5+\frac{7t⋅n}{10}+c⋅n \tag{两边消去 n}\\
t&≤\frac t 5+\frac {7t}{10}+c \tag{再化简}\\
t&≤10c \tag{c 为一个正常数}
\end{align}
$$

其中 c 为一个正常数，故t也是一个正常数，即 $T(n)≤10c⋅n$，因此 $T(n)=O(n)$，至此证明结束。

接下来我们再来探讨下 BFPRT 算法为何选 5 作为分组主元，而不是 2, 3, 7, 9 呢？

首先排除偶数，对于偶数我们很难取舍其中位数，而奇数很容易。再者对于 3 而言，会有 $T(n)≤T(\frac n 3)+T(\frac {2n}3)+c⋅n$，它本身还是操作了 n 个元素，与以 5 为主元的 $\frac {9n}{10}$ 相比，其复杂度并没有减少。对于 7，9，... 而言，上式中的 10c，其整体都会增加，所以与 5 相比，5 更适合。

## 四：参考文献

- 算法导论
- [Median of medians](https://en.wikipedia.org/wiki/Median_of_medians).维基百科
