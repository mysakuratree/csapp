# 2-bomblab

**注：本文是bomb实验速通版。由于bomb实验实在是太火了，网上都是bomb的详细解答，作者做这个实验几次了，所以写writeup的时候不想再用gdb慢慢调试（￣▽￣）。本文用ida速通，适合只想得分的朋友，想慢慢体会该实验的各种精彩细节的，建议转战知乎[手把手教你拆解 CSAPP 的 炸弹实验室 BombLab - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/451623574)**

### 1 简要分析

> 实验目的：熟悉汇编代码，可执行文件调试和栈帧。
>
> 工作空间：可执行文件bomb
>
> 实验内容：从bomb的源代码可以看出，这里一共有7关（有一关隐藏关卡）。每一关有一个输入，然后进入关卡。我们的任务就是通过分析每一关干了什么。
>
> 实验方法：
>
> 1. 动态调试：就是下断点，然后启动gdb直接调试文件，一步一步执行，并时刻关注栈帧变化，动态分析每一关在干什么
> 2. 静态分析：就是直接看汇编代码，理解每一关的逻辑。这个方法很优雅，但可能在函数递归循环调用的时候有些折磨
> 3. 本文方法：IDA大法，直接逆向工程，反汇编为C语言，也算是一种作弊的静态分析

### 2 具体实现

**2.0 bomb源码**

> 分析：可以看到一共七关（有一关隐藏），每关一个输入

![csapp2-1](https://www.hiteva.cn/static/article/csapp/csapp2-1.png)

**2.1 关卡一**

> 分析：IDA秒解

![csapp2-2](https://www.hiteva.cn/static/article/csapp/csapp2-2.png)

> 解答：Border relations with Canada have never been better.

**2.2 关卡二**

> 分析：
>
> 1. 可以看到需要读入6个整数数字
> 2. 关卡二首先检查第一个数是不是1，最后循环判断前一个数是否是后一个数的两倍

![csapp2-3](https://www.hiteva.cn/static/article/csapp/csapp2-3.png)

> 解答：1 2 4 8 16 32

**2.3 关卡三**

> 分析：读入两个整数，前一个数作为switch索引，随便选一个就行，然后后一个数需要等于该索引对应的数

![csapp2-4](https://www.hiteva.cn/static/article/csapp/csapp2-4.png)

> 解答：0 207（或其他几组都可以）

**2.4 关卡四**

> 分析：可以看到也需要读入两个数v3，v4，v3需要小于等于14，然后进入函数func4。

![csapp2-5](https://www.hiteva.cn/static/article/csapp/csapp2-5.png)

> 在func4中，可以看到输入的是a1=v3，a2=0，a3=14，然后在func4中v3=(14 + 0)/2=7，如果第一个数等于7，result=0，直接返回，及给func输入7，0，14，func4返回0。因此我们可以直接输入7，0和func4对应

![csapp2-6](https://www.hiteva.cn/static/article/csapp/csapp2-6.png)

> 解答：7 0

**2.5 关卡五**

> 分析：可以看到需要输入六个字符，然后每个字符取低四位作为索引，在这个array_3449数组中取出六个字符，取完之后的六个字符需要等于flyers。

![csapp2-7](https://www.hiteva.cn/static/article/csapp/csapp2-7.png)

> 在array_3449数组中看到：对应索引为f：9（1001），l：15（1111），y：14（1110），e：5（0101），r：6（0110），s：7（0111）。因此对照下列ASCII码二进制表，取低四位分别为1001，...，0111的即可，及ionefg即可

![csapp2-8](https://www.hiteva.cn/static/article/csapp/csapp2-8.png)

![csapp2-9](https://www.hiteva.cn/static/article/csapp/csapp2-9.png)

> 解答：ionefg

**2.6 关卡六**

> 分析：可以看到也是读入六个数字，设这六个数字存入数组array。
>
> 1. 第一个循环：判断每个数-1后是否大于5，是的话就爆炸，所以这六个数字需要小于等于6，且大于等于1，因为等于0的话，它转为了unsigned，减1会溢出，必然大于5。然后是判断数字两两之间是否相同，相同则爆炸，所以六个数应该都不相同。因此六个数是1，2，3，4，5，6的排列组合
> 

![csapp2-10](https://www.hiteva.cn/static/article/csapp/csapp2-10.png)

> 2. 第二个循环：把六个数分别用7减去，及array[i] = 7 - array[i]

![csapp2-11](https://www.hiteva.cn/static/article/csapp/csapp2-11.png)

> 3. 第三个循环：可以看到，这个node应该是一个链表，每个链表有16个字节，考虑到链表char *类型需要8个字节，还剩下8个字节，而一个int类型为4个字节，因此每个结点中应该是两个int值，一个char *指向下一个结点，可以看到node1到node6的值分别为14Ch=332，0A8h=168，39Ch=924，2B3h=691，1DDh=477，1BBh=443。还剩下一个int值，该值存入的是节点顺序，及该值存入的是13，那该结点就是第三个节点。

![csapp2-12](https://www.hiteva.cn/static/article/csapp/csapp2-12.png)

![csapp2-13](https://www.hiteva.cn/static/article/csapp/csapp2-13.png)

> 4. 第四个循环：可以看到，按照节点顺序依次取出值，和下一个节点比较大小，如果小于下一个节点的值就爆炸，那么可以知道第三个循环需要把结点1到6按照从大到小的顺序排列起来，则按照node中值的大小，顺序应该是3，4，5，6，1，2。按照这个顺序，第三步中依次写入的应该是3，4，5，6，1，2，而经过第二第二步操作逆变，应该是4，3，2，1，6，5

![csapp2-14](https://www.hiteva.cn/static/article/csapp/csapp2-14.png)

> 解答：4，3，2，1，6，5

**隐藏关卡**

> 分析：
>
> 1. 可以看到在每个关卡后面有个phase_defused()函数，点进去看看

![csapp2-15](https://www.hiteva.cn/static/article/csapp/csapp2-15.png)

> 2. 发现有一个输入2位数字，然后跟一个DrEvil字符串，就可以进入隐藏关卡

![csapp2-16](https://www.hiteva.cn/static/article/csapp/csapp2-16.png)

> 3. 那这个是怎么进入呢，可以看到secret_phase()调用了func7，在前面关卡中，只有第四关，调用了一个func4，比较特殊，算是一个暗示，然后我们发现，第四关确实需要输入两个数字，因此尝试在第四关后面加一个DrEvil，就进了隐藏关

![csapp2-17](https://www.hiteva.cn/static/article/csapp/csapp2-17.png)

> 4. 隐藏关读入了一个数字，然后调用func7，并给了两个参数，n1和我们输入的数字，这个n1看栈上看到又是一个结点头，因该是一个链表或者树之类的，看每个结点中有32个字节，有两个8字节的char *类型的东西，那我们可以猜测是一棵二叉树。最后secret_phase()需要func7返回2

![csapp2-18](https://www.hiteva.cn/static/article/csapp/csapp2-18.png)

![csapp2-19](https://www.hiteva.cn/static/article/csapp/csapp2-19.png)

> 5. 看func7，其实func7就是如下代码

```
int func(a, b)
{
	if (a == 0)
	{
		return -1;
	}
	if (a->value > b)
	{
		return 2*func7(a->left_node, b);
	}
	if (a->value != b)
	{
		return 2*func7(a->right_node, b);
	}
	else
	{
		return 0;
	}
}
```

![csapp2-20](https://www.hiteva.cn/static/article/csapp/csapp2-20.png)

> 6. 结合a二叉树中的每个结点的值，反推一下就可以得到输入需要为22
>
> 解答：22

