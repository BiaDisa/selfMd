### 题外话

------------------------

1.java中取模运算规则为

>**A%B == A - (A / B) \* B** 

2.





### 笔记



from- https://www.cnblogs.com/captainad/p/10968103.html 



1.输入一个整数，输出该数二进制表示中1的个数。其中负数用补码表示。

问题思路就是先从高位开始计算1的个数，然后把高位去掉，再计算剩余的，这里用与操作符，同1做&运算，最后剩下的是1。**n&(n-1)的操作，能把n的最高位的1给去掉，以此来计数**。

```Java
1 public static int countOne(int num) {
2     int count = 0;
3     while(num != 0) {
4         num &= (num - 1);
5         count++;
6     }
7     return count;
8 }
```

测试结果：

![](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\1594887398736.png)

实际上结果是相同的，**但是解释是错误的**。

正确解释：

​			n&(n-1)的操作，能把n的最**低**位的1给去掉，以此来计数。

> 如同雪崩一样，会将原数字的最后一个1擦除。







2.判断一个整数是否是2的N次幂

> 这个题目也是当初我面试的时候遇到的，其实很简单，想一下2的N此幂的特征，就是1的N次左移位，在二进制数看来，这个数就是一个1后面带着一串0，很好处理，通过**n&(n-1)**的方式来处理，只要结果是0，那么此数就是2的N次幂。

```
1 public static boolean is2Power(int num) {
2     return (num & (num - 1)) == 0 ? true : false;
3 }
```







## 正题

