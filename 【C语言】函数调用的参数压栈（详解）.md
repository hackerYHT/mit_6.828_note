

# 【C语言】函数调用的参数压栈（详解）

[参考博客](https://blog.csdn.net/muxuen/article/details/123288455)

[TOC]

## 1. 什么是栈区？

栈，是一种[数据结构](https://so.csdn.net/so/search?q=数据结构&spm=1001.2101.3001.7020)。

在学习C语言的过程中，我们一般只关注内存中的3个区域，分别是栈区、堆区和静态区。

其中**堆区主要用于动态内存管理**，在之前的博客中已经和大家介绍过。

![60be7c5869d21aa53b0a38b2987e3826](C:\Users\yingz\Desktop\mit_6.828_note\Pictures\60be7c5869d21aa53b0a38b2987e3826.png)

**而栈区就是编译器给函数运行分配的空间了。**

和堆区空间需要手动分配不同，这一部分空间是编译器自动管理的，函数的栈帧会自动创建，自动销毁。

### 1.1 栈区小知识点

- 栈区的使用是从高地址到低地址
- 栈区的使用遵循先进后出，后进先出
- 栈区的放置是从高地址往低地址放置：`push`压栈
- 删除是从低往高删除：`pop`出栈

# 2.知识点

```c
//本次使用的代码
#include <stdio.h>
int Add(int x, int y)
{
	int z = 0;
	z = x + y;
	return z;
}

int main() 
{
	int a = 10;
	int b = 20;

	int c = Add(a, b);
	printf("%d\n", c);
	
	return 0;
}
```

## 2.1 寄存器

常见寄存器有eax、ebx、ecx、edx，**其中ebp和esp较为特殊**

> ebp、esp这两个寄存器中存放的是地址，这两个地址是用来维护函数栈帧的

- eax/ebx/ecx/edx：通用寄存器，保留临时数据
- ebp：栈低指针
- esp：栈顶指针
- eip：指令寄存器，保存当前指令的下一条指令的地址

------

## 2.2 主函数调用

每一个函数调用，都要在栈区创建一个空间

我们知道`main`函数是程序的入口

实际上，`main`函数也是被其他函数调用的

- `mainCRTStartup`函数调用`__tmainCRTStartup`
- `__tmainCRTStartup`函数调用`main`函数

编译器会先在内存高地址处开辟一部分空间给`mainCRTStartup`和`__tmainCRTStartup`函数，它们进行调用main函数的操作。

![61d575155d9ff67da127253f64553406](C:\Users\yingz\Desktop\mit_6.828_note\Pictures\61d575155d9ff67da127253f64553406.png)

在VS2019中，按F10进行调试，出现黄色小箭头后，`右键-转到反汇编`，即可打开调试中汇编语言的显示界面

![be391aefb4a290bee8f8900278b14176](C:\Users\yingz\Desktop\mit_6.828_note\Pictures\be391aefb4a290bee8f8900278b14176.png)

# 3. 逐条解释

## 3.1 从main开始

先来看第一部分代码，逐条语句进行解释

```c
push  ebp//在栈顶开辟存放ebp这一寄存器对应值的空间
mov  ebp,esp//将esp的值传入ebp中（即将ebp指针移动到原本esp指向的位置）
sub  esp,0E4h//将esp的内容减去0E4h（将esp移动到原esp-0E4h的位置）
push ebx//在栈顶放入ebx
push esi//在栈顶放入esi
push edi//在栈顶放入edi
```

![82b8618cd68b7b40014bf670705ab678](C:\Users\yingz\Desktop\mit_6.828_note\Pictures\82b8618cd68b7b40014bf670705ab678.png)

- lea：load effecticve address 加载有效地址
- dword：double word – 4个字节

```c
lea  edi,[ebp-24h]//将ebp-24h的地址放入edi
mov  ecx,9//将9放入ecx，对应十进制36
mov  eax,0CCCCCCCCh//将0CCCCCCCCh放入eax
rep stos  dword ptr es:[edi]//将edi往下ecx个地址的数据全部初始化为0CCCCCCCCh
1234
```

按F10往下运行，过rep那一步后，可以看到36个字节的数据都被初始化`0CCCCCCCCh`

![d7fa87910cacbb7538f415ccbfa34388](C:\Users\yingz\Desktop\mit_6.828_note\Pictures\d7fa87910cacbb7538f415ccbfa34388.png)

![7ee4e65cf184ec34a04b9d811a88db21](C:\Users\yingz\Desktop\mit_6.828_note\Pictures\7ee4e65cf184ec34a04b9d811a88db21.png)

继续往下运行，可以看到编译器初始化a、b变量的过程

![2d2bdfaef6a9627eac30a6853eb39ff9](C:\Users\yingz\Desktop\mit_6.828_note\Pictures\2d2bdfaef6a9627eac30a6853eb39ff9.png)

![4cbd32835d4a7f1702618db0648243a8](C:\Users\yingz\Desktop\mit_6.828_note\Pictures\4cbd32835d4a7f1702618db0648243a8.png)

- VS2019下是小端存储

  ![6a868923d705abb2deb4fd0d4b9512e1](C:\Users\yingz\Desktop\mit_6.828_note\Pictures\6a868923d705abb2deb4fd0d4b9512e1.png)

```c
	int c =Add(a,b);
mov    eax,dword ptr [b]//把b的内容放入eax
push   eax  //在栈顶放入eax
mov    ecx,dword ptr [a]//把a的内容放入ecx
push   ecx  //在栈顶放入ecx
call   _Add (01A10B4h) //在栈顶放入该地址（call指令下一条指令的地址） 
123456
```

最后这一步`call`很关键，后续会用到

![e9165cad86f471b50535fd31b3f907de](C:\Users\yingz\Desktop\mit_6.828_note\Pictures\e9165cad86f471b50535fd31b3f907de.png)