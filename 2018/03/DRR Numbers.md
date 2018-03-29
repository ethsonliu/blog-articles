前些天遇到一个题目，觉得很有趣。

>A number is a drr number if it is a divisor of its right rotation by one digit.
>
>For example, consider the number 142857, its one-digit-right-rotation is 714285, and we have 714285 = 5 x 142857, so it is a drr number. 
>
>Give you an integer n (1 <= n <= 1000), what is the sum ( mod 100000) of all drr numbers between 10 and 10 ^ n? 

给人的第一感觉就是一个纯数学问题，但怎么也想不到思路，最后在网上找到了一个解决方案。

我们来仔细看下这个例子，可以从中找出一些规律：

![](https://subetter.com/images/figures/20180328_01.png)

- 7 x 5 = 35, next digit is 5 and carry is 3;

- 5 x 5 = 25, plus carry 25 + 3 = 28, next digit is 8 and carry is 2;
- 8 x 5 = 40, plus carry 40 + 2 = 42, next digit is 2, carry is 4;
- 2 x 5 = 10, plus carry 10 + 4 = 14, next digit is 4, carry is 1;
- 4 x 5 = 20, plus carry 20 + 1 = 21, next digit is 1, carry is 2;
- 1 x 5 = 5, plus carry 5 + 2 = 7, next digit would be 7, carry is 0.


我们发现，此时的计算又回到了原点。也就是说，若142857是一个drr number的话，那么142857142857也是一个drr number。

有了这个规律，就很容易编码了。

```c++
#include <iostream>

#define MOD 100000

int digits_to_int(int digits[], int n)
{
	int res = 0;
	int pow10 = 1;

	for (int i = 0; i < n; i++)
	{
		res += (pow10 * digits[i]);
		pow10 *= 10;
	}

	return res;
}

int main()
{
	int n;
	while (~scanf("%d", &n))
	{
		int res = 0;

		for (int multiplier = 2; multiplier < 10; multiplier++)
		{
			for (int least_significant_digit = 1; least_significant_digit < 10; least_significant_digit++)
			{
				bool too_much = false;
				int digits[5] = { least_significant_digit, 0, 0, 0, 0 }; // 最后的结果只取最后 5 位
				int next_digit = 0;
				int carry = 0;
				int digit_index = 1;
				int next_digit_candidate = multiplier * digits[0];

				while (next_digit_candidate != least_significant_digit) // 是否回到起点
				{
					next_digit = next_digit_candidate % 10;

					if (digit_index < 5)
						digits[digit_index] = next_digit;

					carry = next_digit_candidate / 10;
					next_digit_candidate = (multiplier * next_digit) + carry;
					digit_index++;

					if (digit_index > n)
					{
						too_much = true;
						break;
					}
				}

				if (next_digit > 0 && too_much == false) // 剔去前导为 0 的数字
				{
					res += digits_to_int(digits, (digit_index > 5) ? 5 : digit_index) * (n / digit_index);
					res %= MOD;
				}
			}
		}

		// 所有数字都相同的情况
		if (n == 1)
			res += 0;

		if (n >= 2)
		{
			for (int i = 1; i < 10; i++)
			{
				res += 11 * i;
				res %= MOD;
			}
		}

		if (n >= 3)
		{
			for (int i = 1; i < 10; i++)
			{
				res += 111 * i;
				res %= MOD;
			}
		}

		if (n >= 4)
		{
			for (int i = 1; i < 10; i++)
			{
				res += 1111 * i;
				res %= MOD;
			}
		}

		if (n >= 5)
		{
			for (int i = 1; i < 10; i++)
			{
				res += 11111 * i * (n - 4);
				res %= MOD;
			}
		}

		printf("n = %d, the sum (mod 100000) is: %d\n", n, res);
	}
	return 0;
}
```

测试如下：

![](https://subetter.com/images/figures/20180328_02.PNG)

最后有两点需要提示下：

1. 前导0的数字

   例如当multiplier=2，least_significant_digit=1时，我们会得到数字052631578947368421，它同样也符合我们的程序，但这种前导0的数字是应该舍去的，因为它的真实值其实是52631578947368421，而这个值是不符合drr number性质的。


2. multiplier为1的情况

   这很好理解，诸如111，2222，333333，，，，这样所有数字都相同的，都是drr number。

### 参考文献

- [https://github.com/ERufian/RotatedDivisor/blob/master/README.md](https://github.com/ERufian/RotatedDivisor/blob/master/README.md)