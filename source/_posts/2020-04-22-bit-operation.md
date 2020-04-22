---
title: java位运算相关
date: 2020-04-22 15:01:01
tags:
---

## 前言
经常看源码上有些例如 一个int数字的不同位拿来存储多个值，一个int数字表示多个状态之类的。
这篇文章来分析下常用的一些位运算。

## 一些前置概念

左移，右移，原码，反码，补码这些基础运算就不解释了，网上一大堆。
这一篇还比较详细
https://www.cnblogs.com/zhangziqiu/archive/2011/03/30/computercode.html

|  操作符   | 描述  | 例子  |
|  ----  | ----  |----  |
|`按位与&` | 如果相对应位都是1，则结果为1，否则为0 | `1&3=1`，即0001&0011 =0001
| <code>按位或&#124;</code>| 如果相对应位都是0，则结果为0，否则为1 | <code>1&#124;3=3</code>，即0001&#124;0011 =0011
| 按位异或`^` |相对应位一个0，一个1，则结果1，否则0 | `1^3=2`，即0001^0011=0010
| 按位取反`~` |按位取反运算符翻转操作数的每一位，即0变成1，1变成0 |
| 左移 `<<` |按位左移运算符。左操作数按位左移右操作数指定的位数 |1 << 2 = 4，即0100
| 右移 `>>`| 按位右移运算符。左操作数按位右移右操作数指定的位数| 4 >> 2 = 1，0100 >> 2 = 0001

## 具体例子

### 按位存储不同的数值
看一段线程池原码，我们通过注解来分析
```java
public class ThreadPoolExecutor extends AbstractExecutorService {
    //这里其实就是保存了2个数值，低位和高位分别保存为两块领地，各自存储数据。
    private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));

    // 这里定义一个界限，定义了一个integer字节的高3位和剩下的位两个领地,高3位存储的时候要跳过低位的领地
    private static final int COUNT_BITS = Integer.SIZE - 3;

    /**
    * 这东西是给你来解码(取数据)用的, 1左移29位 = 00100000000000000000000000000000
    * 再减去 1 = 00011111111111111111111111111111
    * 这里刚好 高3位 为0，剩下的位为1,高低区域路径分明。
    **/
    private static final int CAPACITY   = (1 << COUNT_BITS) - 1;
    // Packing and unpacking ctl
    /**
    * 通过 c & CAPACITY 来取出低位的值(高3位都是0， &操作的时候 c的高3位就直接忽略)
    * 通过 c & (~CAPACITY) 来取出高位的值(取反之后高3都是1，低29位是0 ，c的低29位就忽略了)
    **/
    private static int runStateOf(int c)     { return c & ~CAPACITY; }
    private static int workerCountOf(int c)  { return c & CAPACITY; }

    // runState is stored in the high-order bits
    // -1 的补码是 11111111111111111111111111111111 左位移 29位，左位移右边补0. 那么高3 = 111，是个负数
    private static final int RUNNING    = -1 << COUNT_BITS;
    // 0怎么移动，所有位都是0，那么高3 = 000
    private static final int SHUTDOWN   =  0 << COUNT_BITS;

    // 下面三个都是正数(因为符号位是0)
    // 这里一样的计算 高3 = 001 
    private static final int STOP       =  1 << COUNT_BITS;
    // 高3 = 010
    private static final int TIDYING    =  2 << COUNT_BITS;
    // 高3 = 011
    private static final int TERMINATED =  3 << COUNT_BITS;

    // 通过或操作让低位和高位 二进制有数值的都同时保留。
    private static int ctlOf(int rs, int wc) { return rs | wc; }
}
```

### 想一个字段存多个值。

类似于多选框，用户勾选了多少个状态字段。
比较常见的用字符串，逗号分隔。`"1,2,3,4"`

如果了解位运算，状态不是很多 这里也可以用位运算表示,一个整数代替多个状态

`11111111111111111111111111111111`

按int=32位来说，可以把每一位代表一种状态的选中(1)和非选中(0)，为1代表这个状态位有选中，0代表没选。
一个int可以同时保存32个状态是否勾选的信息。

每一位代表这个位对应的一个状态，如果这32位有多个1，会代表多个状态，
每个独立的状态对应的数值都是1，其他是0的有 
1<<0,1<<1,1<<2,1<<3,1<<4,.......1<<31 ,这32个
你只能存储这32个字段，其他数字代表的都是这32个状态的1组合。

例如 1和2，我们都选中的话 就是 0001|0010 = 0011 = 3;
0011 又能拆出 0001 + 0010 2个独立的状态。

一段例子来看。

```java
public class TypeAndSize {
    private static final int status_1 = 1 << 0;
	private static final int status_2 = 1 << 1;
	private static final int status_3 = 1 << 2;
	private static final int status_4 = 1 << 3;

    private List<Integer> bitToList(int num) {
        List<Integer> result = new ArrayList<>();
        for (int bit = 0; bit < Integer.SIZE; bit++) {
            // 从第0位开始左移，这个数值当基准点一直判断到最大位
            int referenceValue = 1 << bit;
            // 这里数值都是正整数来讲的，基准点是左移递增的，如果基准点都大于实际存储的数字，说明num之后的高位都0不用对比了
            // 如果小于0说明最高位32位肯定有数值，那么就不能跳出循环，只能对比全部位置了。
            if ( num > 0 && referenceValue > num) {
                break;
            }
            // 依次判断这个位有没有1， 如果有1的话 这个等式肯定相等，因为referenceValue其他位都是0
            if ((referenceValue & num) == referenceValue) {
                result.add(referenceValue);
            }
        }
        return result;
    }

    private int listToBit(List<Integer> list) {
        //  | 代表有一个位子是1 都是1  ，把list里面所有传递过来的状态都 合并起来,
        return list.stream().reduce(1, (a, b) -> a | b);
    }
    public static void main(String[] args) {
        TypeAndSize obj = new TypeAndSize();

        // 将多个状态合并成一个值
        int status = obj.listToBit(Arrays.asList(status_1,status_2,status_3,status_4));

        // 取出存储的都有哪些状态
        obj.bitToList(status).forEach(System.out::println);
    }
}
```


