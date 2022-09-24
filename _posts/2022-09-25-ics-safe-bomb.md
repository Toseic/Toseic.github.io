---
layout: post
title: "拆弹日记-如何制作本地炸弹2022版"
author: "Toseic"
tags: ics bomblab trick
comments: true
excerpt_separator: <!--more-->
---
这篇文章参考了： [https://infedg.xyz/index.php/archives/35/](https://infedg.xyz/index.php/archives/35/)

想必大家已经看过了去年的炸弹本地化方法，但是今年并不适用了，所以我们接着研究如何制作本地化的炸弹。
先看看`bomb.c`的这部分:

```c
    int *fp = initialize_bomb();

    if (*fp != SECRETTOKEN){
      printf("Don't try to make the bomb run on your local machine!(*/w＼*)");
      return 0;
    }
```

逻辑是首先获取`*fp`，然后与`SECRETTOKEN`进行比较。

所以我们的思路：直接绕过`initialize_bomb`，然后把后面与比较`SECRETTOKEN`的这部分绕过。

## 绕过`initialize_bomb`

依然是熟悉的配方：找到`initialize_bomb`的起始位置:

```assembly
0000000000001d31 <initialize_bomb>:
    1d31:	f3 0f 1e fa          	endbr64 
    1d35:	55                   	push   %rbp
    1d36:	53                   	push   %rbx
```

然后再打开`hexedit`，把`1d31`（不同的炸弹可能不一样，下同）的位置改成`C3` （ `retq` 的机器代码）。

*如何使用hexedit可以参考篇首的文章，或者自行在网上查找方法。*



## 绕过与`SECRETTOKEN`的比较

我们可以先想想`SECRETTOKEN`是怎么获取的，我一开始以为是从服务器获取的，但是后来发现：

```assembly
    1de2:	e8 49 f5 ff ff       	callq  1330 <malloc@plt>
    1de7:	c7 00 11 fa 22 20    	movl   $0x2022fa11,(%rax) ;将token加入内存
    1ded:	48 8b 8c 24 48 20 00 	mov    0x2048(%rsp),%rcx
```

发现`SECRETTOKEN`是直接添加的，事情便简单了很多，我们先找到比较的那部分：

```assembly
    151c:	48 89 c3             	mov    %rax,%rbx
    151f:	81 38 11 fa 22 20    	cmpl   $0x2022fa11,(%rax) ;比较token是否正确
    1525:	74 72                	je     1599 <main+0xb0>
```

一个简单的绕过的思路是把`cmpl   $0x2022fa11,(%rax)`换成一个永远为真的式子`cmp %ebx,%eax`*3，也就是`39D839D839D8`，重复三遍是为了占满6个字节，（当然其他恒为真的比较式应该都可以）。



## 结尾

感觉“制作出一个本地炸弹”似乎已经成为了`bomblab`的另一个隐藏关卡，似乎每年都有类似的这种文章hhhhh。
最后盲猜一手，也许明年的bomb会改成从服务器获取`SECRETTOKEN`(?)

