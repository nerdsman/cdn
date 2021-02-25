---
title: C语言提高
tags:
  - C
  - 编程
categories: 转载
abbrlink: 59864
date: 2021-02-03 15:18:43
---
## C核心编程

## 1  前言

### 1.1 听课要求

- 专心听讲、积极思考；
- 遇到不懂的暂时先记下，课后再问；
- 准备一个笔记本，或者笔记软件（记事本、word、typroa、印象笔记等）
- 敲代码的时候一定要动手去干，不动手，永远学不会；
- 杜绝边听边敲、杜绝犯困听课。
- 如果时间允许，请课前做好预习；
- 尽量少回看视频，别对视频产生依赖，可以用2倍速度回看视频；
- 按时完成老师布置的练习，根据自己的理解总结学到的知识点；
- 初学者 应该抓住重点，不要钻牛角尖遇到问题了，优先自己尝试解决，其次谷歌百度，最后再问老师；
- 如果时间允许，可以多去网上找对应阶段的学习资料面试题，
- 注意作息，注意劳逸结合，积极锻炼。





### 1.2 技术层次

| 基础     | 提高       | 实践  |
| -------- | ---------- | ----- |
| C提高    | Qt界面编程 | 项目1 |
| 数据结构 | Linux编程  | 项目2 |
| C++      | 数据库编程 | 项目3 |

 



## 2 内存分区

### 2.1 内存区域

C/C++编译的程序占用的内存分为以下几个区域

* 代码区
* 全局区/静态区
* 栈区
* 堆区





**划分：** 

​	程序运行前： 代码区、全局区/静态区

​	程序运行后：栈区、堆区



### 2.2  内存四区

#### 2.2.1  代码区

**作用：**存放CPU执行的二进制机器指令

**特点：** 

1. 只读
2. 共享



#### 2.2.2 栈区

**特点：**

* 栈是一种先进后出的内存结构，由编译器自动分配释放数据。
* 主要存放函数的形式参数值、局部变量等。
* 函数运行结束，相应栈变量会被自动释放
* 栈空间较小，不适合将大量数据存放在栈中



**总结：**

管理方式：编译器自动管理该区内存。
空间大小：提前规定、较小。
生命周期：函数使用完毕立即释放。


**注意事项：**

* 不要返回局部变量的地址

```C
//栈区上开辟的数据由系统进行管理，不需要程序员管理开辟和释放
int * func()
{
	int a = 10;
	return &a;
}
//不管结果是否正确，这个值已经被释放了，不可以操作一块非法的内存空间
void test01()
{
	int * p = func();

	printf("a = %d\n", *p);
	printf("a = %d\n", *p); 

}
```



```C
char * getString()
{
	char str[] = "hello world";
	return str;
}

void test02()
{
	char * p = NULL;
	p = getString();

	printf("p = %s\n", p);
}
```





#### 2.2.3 堆区

##### 2.2.3.1 堆区基本概念

**特点：**

* 堆区由开发人员手动申请和释放，在释放之前，该块堆空间可一直使用。
* 由程序员分配和释放，若程序员不释放，程序结束时由系统回收内存。
* 堆空间一般没有软限制，只受限于硬件。会比栈空间更大，适宜存放较大数据。



**总结：**

管理方式：开发人员手动申请和释放。
空间大小：较大。
生命周期：手动释放之前一直存在，或程序结束由系统回收。



**堆区使用：**

```C
int * getSpace()
{
	int * p =  malloc(sizeof(int)* 5);
	if (p == NULL)
	{
		return;
	}
	for (int i = 0; i < 5;i++)
	{
		p[i] = i + 100; //所有在连续空间上开辟的数据，都可以利用下标进行索引
	}
	return p;
}

void test01()
{
	int * p = getSpace();
	for (int i = 0; i < 5;i++)
	{
		printf("%d\n", p[i]);
	}
	//堆区的数据  自己开辟，自己管理释放
	free(p);
	p = NULL;
}
```



##### 2.2.3.2 堆区注意事项

**注意事项：**

主调函数中没有给指针分配内存，被调函数需要利用高级指针进行分屏

分析以下代码运行的结果

```C
void allocateSpace(char * pp)
{
	char * temp = malloc(100);
	memset(temp, 0, 100);
	strcpy(temp, "hello world");
	pp = temp;
}

void test02()
{
	char * p = NULL;
	allocateSpace(p);
	printf("%s\n", p);
}
```

**解决方式1**

```C
//利用高级指针
void allocateSpac2(char ** pp)
{
	char * temp = malloc(100);
	memset(temp, 0, 100);
	strcpy(temp, "hello world");
	*pp = temp;
	printf("aaa%s\n", *pp);
}

void test03()
{
	char * p = NULL;
	allocateSpac2(&p);
	printf("%s\n", p);
}
```

**解决方式2**

```C
//利用返回值
char* allocateSpace3()
{
	char *temp = malloc(100);
	memset(temp, 0, 100);

	strcpy(temp, "hello world");
	return temp;
}
void test04()
{
	char *p = NULL;
	p = allocateSpace3();
	printf("%s\n", p);
}
```



##### 2.2.3.3 calloc和realloc

内存申请我们可使用三个函数来完成，分别为：malloc、calloc、realloc，内存释放我们只需要使用 free 函数。 

1. malloc 函数:
   原型：`void *malloc(unsigned int num_bytes)` 
   用法：分配长度为 num_bytes 字节的内存块。
   说明：如果分配成功则返回指向被分配内存的指针，否则返回 NULL。
2. calloc 函数:
   原型：`void *calloc(int num_elems, int elem_size)`
   用法：为具有 num_elems 个长度为 elem_size 元素的数组分配内存。
   说明：如果分配成功则返回指向被分配内存的指针，否则返回 NULL。
3. realloc 函数:
   原型：`void *realloc(void *mem_address, unsigned int newsize)`
   作用：改变 mem_address 所指内存区域的大小为 newsize 长度。
   说明：如果重新分配成功则返回指向被分配内存的指针，否则返回 NULL。
4. free 函数:
   原型：`void free(void *p);`
   作用：释放指针 p 所指向的的内存空间。
   说明：p所指向的内存空间必须是用 calloc,malloc,realloc 所分配的内存。如果 p 为 NULL则不做任何操作。

**calloc案例：**

```C
//1. calloc
void test01()
{
	int *p =  calloc(10,sizeof(int));  
	for (int i = 0; i < 10; ++i)
	{
		p[i] = i + 1;
	}
	for (int i = 0; i < 10; ++i)
	{
		printf("%d\n", p[i]);
	}
	if (p != NULL)
	{
		free(p);
		p = NULL;
	}
}
```



**realloc案例：**

```C
void test02()
{
	int *p = malloc(sizeof(int)* 10);
	for (int i = 0; i < 10; ++i)
	{
		p[i] = i + 1;
	}
	for (int i = 0; i < 10; ++i)
	{
		printf("%d ", p[i]);
	}
	printf("%d\n", p);
    
	p = realloc(p, sizeof(int)* 200);   
	printf("%d\n",p);

	for (int i = 0; i < 15; ++i)
	{
		printf("%d ", p[i]);
	}
}
```



#### 2.2.4 全局/静态区

##### 2.2.4.1 基本概念

**特点：**

* 全局/静态区存储**全局变量、静态变量、常量**，该区变量在程序运行期间一直存在
* 程序结束由系统回收。
* 已初始化的数据放在data段，未初始化的数据放到bss段
* 该区变量当未初始化时，会有有默认值初始化。



**总结：**

管理方式：编译器自动管理该区内存。
生命周期：程序结束释放。



##### 2.2.4.2 全局变量

```C
//其他文件中声明第二行代码
extern int g_a = 10; //c语言中 默认全局变量前 加了关键字 extern

void test01()
{
	extern int g_a;
	printf("g_a = %d\n", g_a);
}
```



```C
extern int g_b;
void test02()
{
	printf("g_b = %d\n", g_b);
}
```



##### 2.2.4.3 静态变量

```C
void func()
{
	static int s_a = 10; //静态变量只初始化一次
	s_a++;
	printf("%d\n", s_a);
}

void test03()
{
	func();
	func();
	func();
}
```



```C
void test04()
{
	static int s_a; //如果未初始化，默认为0
	printf("%d\n", s_a);
}
```



##### 2.2.3.4 常量

1. const修饰的变量
2. 字符串常量



**const修饰的变量：**

```C
//全局常量
const int a = 10; //全局常量存放到常量区，收到常量区的保护

void test01()
{
    //a = 20; //直接修改失败
	int * p = &a;
	*p = 30;  //间接修改 语法通过，运行失败
	printf("a = %d ", a); 


	//局部常量
	const int b = 10; //b分配到了栈上,可以通过间接方式对其进行修改
	//b = 30; //直接修改失败
	int * p2 = &b;
	*p2 = 30;
	printf("b = %d\n", b); //间接修改成功，C语言下const修饰的局部常量为伪常量
}

```

**结论：** 

​	全局修改失败，局部修改成功



**字符串常量：**

```C
//1、字符串常量 是可以共享的
void test01()
{
	char * p1= "hello world";
	char * p2 = "hello world";
	char * p3 = "hello world";
	printf("%d\n",&"hello world");
	printf("%d\n", p1);
	printf("%d\n", p2);
	printf("%d\n", p3);
}

//2、vs下 不可以修改字符串常量中的内存
void test02()
{
	char * p1 = "hello world";
	printf("%d\n", p1);
	printf("%c\n", p1[0]);
	//p1[0] = 'W'; //不允许修改 常量区内容
}
```



上述案例中字符串常量修改失败，但这不是绝对的

* ANSI C中规定：修改字符串常量，结果是未定义的
* 有些编译器把多个相同的字符串常量看成一个（节省空间），有些则不进行此优化
* 有些编译器可修改字符串常量，有些编译器则不可修改字符串常量

**结论： 尽量不要去修改字符串常量**







### 2.3 函数调用模型

#### 2.3.1 宏函数

**宏函数概念：**

* 宏函数和宏常量都是利用#define定义出来的内容
* 在项目中，经常把一些短小而又频繁使用的函数写成宏函数
* 这是由于宏函数没有普通函数参数压栈、跳转、返回等时间上的开销，可以调高程序的效率。



**注意事项：** 宏函数通常需要加括号，保证运算的完整



```C
#define MYADD(x,y)  ((x) + (y)) //不是函数 ，宏函数

//普通函数下的a、b都要进行入栈，函数执行后出栈
int  myAdd(int a ,int b)
{
	return a + b;
}
//宏函数  在一定的场景下  要比普通的函数效率高，把频繁使用并且短小的函数 可以写成宏函数
//宏函数在编译阶段就替换源码
//而没有普通函数入栈出栈的开销，以空间换时间
void test01()
{
	int a = 10;
	int b = 20;
	printf("a + b = %d\n", MYADD(a, b)); //  ((a) + (b))
}
```

**总结：**将频繁短小的函数可以封装为宏函数，以空间换时间。



#### 2.3.2 函数调用流程

在2.3.1中我们学会了宏函数，其中讲到了普通函数执行的时候会有些时间的开销

下面我通过以下案例来分析下 ，一个简单的函数调用流程

```C
int func(int a,int b){
	int t_a = a;
	int t_b = b;
	return t_a + t_b;
}

int main(){
	int ret = 0;
	ret = func(10, 20);
	return EXIT_SUCCESS;
}
```

通过上面的案例，我们知道一个普通函数的调用流程

在此期间又引出两个问题

1. 被调函数func中的a、b入栈的顺序是从左到右，还是从右到左？
2. 当被调函数执行完毕后，a、b这两个参数是由主调函数mian去管理释放还是被调函数func管理释放？

带着这两个问题来看**2.3.3 调用惯例**



#### 2.3.3 调用惯例

**调用惯例概念：**

函数的调用方（主调函数）和被调用方（）对于函数是如何调用的必须有一个明确的约定，

只有双方都遵循同样的约定，函数才能够被正确的调用，这样的约定被称为**调用惯例**



C/C++语言中存在多个调用惯例，默认使用的调用惯例为 **cdecl** 

**注: cdecl不是标准的关键字，在不同的编译器里可能有不同的写法，例如gcc里就不存在_cdecl这样的关键字**


调用惯例包含的内容有：

| 调用惯例 | 出栈方     | 参数传递                             | 名字修饰                   |
| -------- | ---------- | ------------------------------------ | -------------------------- |
| cdecl    | 函数调用方 | 从右至左参数入栈                     | 下划线+函数名              |
| stdcall  | 函数本身   | 从右至左参数入栈                     | 下划线+函数名+@+参数字节数 |
| fastcall | 函数本身   | 前两个参数由寄存器传递，后面从右到左 | @+函数名+@+参数的字节数    |
| pascal   | 函数本身   | 从左至右参数入栈                     | 较为复杂，参见相关文档     |



#### 2.3.4 栈的生长方向



**作用：**了解栈中元素与元素之间的存储方式

```C
//栈的生长方向   自上而下  ，栈底高地址  栈顶低地址
void test01()
{
	int a = 10;
	int b = 20;
	int c = 30;
	int d = 40;

	printf("%d\n", &a); 
	printf("%d\n", &b);
	printf("%d\n", &c);
	printf("%d\n", &d);
}
```

**结论：**栈底高地址  栈顶低地址





#### 2.3.5 内存存储方式

内存中的多字节数据相对于内存地址有大端和小端之分。



小端模式：==高==位字节数据保存在内存的==高==地址中，==低==位字节数据保存在内存的==低==地址中

大端模式：==高==位字节数据保存在内存的==低==地址中，==低==位字节数据保存在内存的==高==地址中



**验证方式：**

```C
	int a = 0x11223344;
	char * p = &a; //char * 改变指针步长，一次跳一个字节 

	printf("%x\n", *p);      // 44  低地址  -- 低位字节
	printf("%x\n", *(p+1));
	printf("%x\n", *(p+2));
	printf("%x\n", *(p+3));  // 11  高地址  --  高位字节
```





## 3 指针进阶

### 3.1 空指针和野指针

#### 3.1.1 空指针

​	含义：特殊的指针变量，表示不指向任何东西

​	作用：指针变量创建的时候，可以初始化为NULL

​	注意：不要对空指针进行解引用操作



#### 3.1.2 野指针

​	含义：野指针指向一个已释放的内存或者未申请过的内存空间

​	注意：不要对野指针进行解引用操作



```C
void test01(){
	char *p = NULL;
	//给p指向的内存区域拷贝内容
	strcpy(p, "1111"); //err

	char *q = 0x1122;
	//给q指向的内存区域拷贝内容
	strcpy(q, "2222"); //err		
}
```



#### 3.1.3 常见的野指针

1. 指针变量为初始化
2. 释放堆区的指针后未置空
3. 指针操作超越了变量作用域

```C
int* func()
{
	int a = 10; 
	int * p = &a; 
	return p;
}

void test02()
{
	//1 指针变量未初始化
	int * p1;
	//printf("%d\n", *p1);

	//2 指针释放后未置空
	int * p2 = (int*)malloc(sizeof(int));
	*p2 = 100;
	free(p2);
	printf("%d\n", *p2); //乱码 已经释放了

	//3 指针操作超越变量作用域
	int * p3 = func();
	printf("%d\n", *p3);
}
```

注意： `空指针可以free，但是野指针不可以`





### 3.2 指针步长的意义

作用：清楚不同类型的指针有何不同的意义

* 指针变量+1后跳跃字节数量不同
* 解引用的时候，取出字节数量不同

```C
void test01()
{
	char * p = NULL;
	printf("%d\n", p);
	printf("%d\n", p+1);

	int * p1 = NULL;
	printf("%d\n", p1);
	printf("%d\n", p1 + 1);
}

//2、解引用时候  取多少字节数
void test02()
{
	char buf[1024] = { 0 }; //1024字节
	int a = 1000;  //4字节
	memcpy(buf+1, &a, sizeof(int));
	char * p = buf;
	printf("%d\n",  *(int*)(p+1) );
}
```



练习：在下面结构体中利用地址偏移，找到结构体中d属性的位置以及打印该属性

```C
struct Person
{
	char a; 0 ~ 3
	int b;  4 ~ 7
	char buf[64];  8 ~ 71
	int d;   72 ~ 75
};

void test03()
{
	struct Person  p = { 'A', 10, "hello world", 10000 };

	//自定义数据类型找偏移量 可以通过函数寻找
	printf("%d\n", offsetof(struct Person, d));//#include <stddef.h>

	//打印d的数字
	printf("d = %d\n", *(int*)((char *)&p + offsetof(struct Person, d)));
}
```



### 3.3 指针的间接赋值

变量的修改方式有两种：直接修改、间接修改

通过指针可以进行间接赋值，间接赋值成立的条件为：

1. 两个变量（普通变量+指针变量） / (实参+形参)
2. 建立关系
3. 通过*操作指针指向的内存



```C
void changeValue(int *p)
{
	*p = 1000;
}

void test01()
{
	//1 、一个普通变量 和一个指针变量  构成指针的间接赋值
	int a = 10;
	int * p = &a;
	*p = 100;
	printf("a = %d\n", a);

	//2、通过实参和形参 进行间接赋值
	int a2 = 0;
	changeValue(&a2);
	printf("a2 = %d\n", a2);
}
```

思考：如果我们可以提前知道变量的地址编号，是否也可以修改内存空间？



### 3.4 指针做函数参数输入输出特性

输入特性：主调函数分配内存，被调函数使用

输出特性：被调函数分配内存，主调函数使用



#### 3.4.1 输入特性

```C
//栈区分配
void fun(char *p)
{
	//给p指向的内存区域拷贝内容
	strcpy(p, "helloabcde");
}

void test02(void)
{
	//栈上分配内存
	//输入，主调函数分配内存
	char buf[1024] = { 0 };
	fun(buf);
	printf("buf  = %s\n", buf);
}

//堆区分配
void printString(char * str)
{
	printf("%s\n", str);  
}
void test01()
{
	//堆区分配内存
	char * p = malloc(sizeof(char)* 64);
	memset(p, 0, 64);
	strcpy(p, "hello world");
	printString(p);
}
```



### 3.4.2 输出特性

```C
void allocteSpace( char  ** p)
{
	char  * tempP =  malloc(sizeof(char)* 64);

	memset(tempP, 0, 64);
	strcpy(tempP, "hello world!!!");
	*p = tempP;

}

void test03()
{
	char *p = NULL;
	allocteSpace(&p);
	printf("%s\n", p);
	if (p!=NULL)
	{
		free(p);
	}
	p = NULL;
}
```





### 3.5 指针易错点

#### 3.5.1 指针越界

```C
void test01(){
	char buf[8] = "zhangtao";
	printf("buf:%s\n",buf);
}
```



#### 3.5.2 返回局部变量地址

```C
char *getString()
{
	char str[] = "abcdedsgads"; //栈区，
	printf("getString - str = %s\n", str);
	return str;
}
void test02()
{
    char * str = getString();
    printf("test - str = %s\n", str);
}
```



#### 3.5.3 同一块内存释放多次

```C
void test03()
{
    char *p = malloc(sizeof(char) * 64);
	free(p);
	free(p);
}
```





#### 3.5.4 释放偏移后的指针

```C
void test04()
{
	char str[] = "hello world";
	char *p = malloc(64);
	for (int i = 0; i <= strlen(str); i++)
	{
		*p = str[i];
		++p;
	}
	free(p);
}
```





### 3.6 二级指针输出输出特性

#### 3.6.1 二级指针输入特性

```C
void printArray(int **arr , int len)
{
	
	for (int i = 0; i < 5;i++)
	{
		printf("数组中第%d个元素的值为%d \n", i + 1, *arr[i]); //值
	}
}

//堆区开辟空间
void test01()
{
	int ** pArray = malloc(sizeof(int *)* 5);

	//在栈中分配数据
	int  a1 = 100;
	int  a2 = 200;
	int  a3 = 300;
	int  a4 = 400;
	int  a5 = 500;

	pArray[0] = &a1;
	pArray[1] = &a2;
	pArray[2] = &a3;
	pArray[3] = &a4;
	pArray[4] = &a5;

	int len = 5; 
	printArray(pArray, len);

	//释放堆区空间
	if (pArray != NULL)
	{
		free(pArray);
		pArray = NULL;
	}

}


//在栈上开辟空间
void test02()
{
	int * pArray[5]; //开辟到栈中

	for (int i = 0; i < 5;i++)
	{
		pArray[i] = malloc(4);
		*(pArray[i]) = i + 100;
	}

	printArray(pArray, 5);

	//释放堆区空间
	for (int i = 0; i < 5;i++)
	{
		if (pArray[i] != NULL)
		{
			free(pArray[i]);
			pArray[i] = NULL;
		}
	}
}
```



#### 3.6.2 二级指针输出特性

```C
void allocateSpace(int ** p)
{
	int *arr = malloc(sizeof(int)* 10);
	for (int i = 0; i < 10;i++)
	{
		arr[i] = i;
	}
	*p = arr;
}

void printArray(int ** arr, int len)
{
	for (int i = 0; i < len;i++)
	{
		printf("%d\n", (*arr)[i]);
	}
}

void freeSpace(int ** arr)
{
	if (*arr!=NULL)
	{
		free(*arr);
		*arr = NULL;
	}
}

void test01()
{
	int * p = NULL;
	allocateSpace(&p);


	printArray(&p, 10);

	freeSpace(&p);

	if (p == NULL)
	{
		printf("数组指针已经置空\n");
	}
}
```



## 4 位运算



### 4.1 数据的存储方式



#### 4.1.1 printf格式化输出

作用：利用printf输出不同进制下的数字

```C
void test01()
{
	int a = 10;

	printf("十进制：%d\n", a);
	printf("八进制：%#o\n", a);
	printf("十六进制：%#x\n", a);
	printf("十六进制：%#X\n", a);
}

void test02()
{
	int a = 123;		//十进制方式赋值
	int b = 0123;		//八进制方式赋值， 以数字0开头
	int c = 0xabc;	    //十六进制方式赋值

	printf("十进制：%d\n", a);
	printf("八进制：%#o\n", b);	//%o,为字母o,不是数字
	printf("十六进制：%#x\n", c);
}
```



#### 4.1.2  存储方式

##### 4.1.2.1存数据

在计算机中，数据有三种表现形式，分别为：原码、反码、补码

* 原码：最高位为符号位，0代表正，1代表负
* 反码：无符号或正数，原码 = 反码；负数反码 = 原码符号位不变，其余为取反
* 补码：无符号或正数，原码 = 反码； 负数补码 = 反码+1

**注：计算机存放数据都是用补码形式**



**总结：**

* 无符号数以及有符号数的正数  源码 = 反码 = 补码
* 符号数 负数  反码 = 原码 取反（不包括符号位） 补码 = 反码 + 1  



##### 4.4.2.2 补码意义

* 统一了零的编码

| **十进制数** | **原码**  |
| ------------ | --------- |
| +0           | 0000 0000 |
| -0           | 1000 0000 |

| **十进制数** | **反码**  |
| ------------ | --------- |
| +0           | 0000 0000 |
| -0           | 1111 1111 |

| **十进制数** | **补码**                                              |
| ------------ | ----------------------------------------------------- |
| +0           | 0000 0000                                             |
| -0           | 10000 0000由于只用8位描述，最高位1丢弃，变为0000 0000 |



* 将减法运算转变为加法运算

  例如： 9 - 6

| **十进制数** | **原码**  |
| ------------ | --------- |
| 9            | 0000 1001 |
| -6           | 1000 0110 |

如果用原码计算：

| 0000 1001   |
| ----------- |
| 1000 0100 + |
| 1000 1111   |

结果： 9 - 6 = -15不正确



利用补码进行计算

| **十进制数** | 补码       |
| ------------ | ---------- |
| 9            | 0000  1001 |
| -6           | 1111  1010 |

| 0000 1001   |
| ----------- |
| 1111 1010 + |
| 10000 0011  |

最高位的1溢出，剩余8位二进制表示为3，结果正确



##### 4.1.2.3 取数据

1. 无符号取数据： %u %lu   %llu  %o  %x  ...
2. 有符号取数据： %d  %ld  %lld  %f  %llf ...



取数据步骤：

* 高位如果是0， 代表正数   原码 = 反码 = 补码 ，原样输出
* 高位如果是1， 代表负数  符号位不变，其余位取反 + 1



注：程序中的八进制和十六进制的数据，不用考虑正负，按照无符号对待

相关案例：



```C
void test01()
{
	char num = -15;
	//原码  1000 1111
	//反码  1111 0001
	//补码  1111 1010
    
	//有符号输出 
	printf("%d\n", num); 

	//无符号输出
	printf("%u\n", num & 0x000000ff);
	//编译器自动让结果与 前3个字节为1 的数字 做了按位或操作
	/*
		0000 0000 0000 0000 0000 0000 1111 0001  反码
		1111 1111 1111 1111 1111 1111 0000 0000 |
		1111 1111 1111 1111 1111 1111 1111 0001  结果
	*/
}

void test02()
{

	char num = 0x9b;  // 这个就是 1001  1011  就是补码 
	//计算机原样存储  
	// 有符号取
	// 补码 1001  1011
	// 原码 1110  0101   -（64 + 32 + 4 + 1） = - 101
	printf("%d\n", num);

	//无符号取
	printf("%x\n", num & 0x000000ff); //9b
	printf("%u\n", num & 0x000000ff); // 128 + 16 + 8 + 2 + 1 = 155
}
```





### 4.2 位运算

#### 4.2.1 位运算符

1. 按位取反 ==~==，对每个二进制位进行取反，0变1,1变0
2. 位与 ==&==，   同真为真，其余为假
3. 位或 ==|==，    同假为假，其余为真
4. 位抑或 ==^==，相同为假，不同为真

```C
//1. 按位取反 ~
void test01()
{
	int number = 2;
	printf("~number : %d\n", ~number);

	int num = -2;
	printf("%d\n", ~num); 
}

//2. 位与 &
void test02()
{
	int number = 332;
	if ((number & 1) == 0)
	{
		printf("%d是偶数!\n",number);
	}
	else
	{
		printf("%d是奇数!\n",number);
	}

	//number = number & 0;
	number &= 0;
	printf("number:%d\n", number);
}

//3. 位或 |
void test03()
{
	int num1 = 5;
	int num2 = 3;

	printf("num1 | num2 = %d\n", num1 | num2);
}

//4 位抑或 ^
void test04()
{
	int num1 = 5;
	int num2 = 10;
    
	printf("num1:%d num2:%d\n", num1, num2);

	num1 = num1 ^ num2;
	num2 = num1 ^ num2;
	num1 = num1 ^ num2;

	printf("num1:%d num2:%d\n",num1,num2);
}
```





#### 4.2.2 移位运算符

1. 左移 <<，按二进制形式把所有的数字向左移动对应的位数，高位移出(舍弃)，低位的空位补 0。
2. 右移 >>，按二进制形式把所有的数字向右移动对应位移位数，低位移出(舍弃)，高位的空位补符号位

```C
//左移运算符 左移几位就相当于乘以2的几次方  
void test05()
{
	int number = 20;
	printf("number = %d\n", number <<= 2);
	printf("number = %d\n", number >>= 1);
}
```

注： 右移运算，对于无符号数据，高位用0填补，有符号正数用0，负数有些计算机系统用0，有些用1，不是绝对





## 5 数组进阶

### 5.1 一维数组与数组名

数组定义：一片连续内存空间里，放相同类型的数据元素

思考：一维数组的数组名是不是指向数组中第一个元素的指针？



```C
void printArray( int arr[] , int len) // int arr[] ====  int * arr 前者可读性高
{
	for (int i = 0; i < len;i++)
	{
		printf("%d\n", arr[i]); 
	}
}

void test01()
{

	int arr[5] = { 1, 2, 3, 4, 5 };

	//1、当sizeof数组名时候，统计是整个数组的大小
	printf("sizeof(arr) = %d\n", sizeof(arr));

	//2、当对数组名 取地址的时候
	printf("%d\n", &arr);
	printf("%d\n", &arr + 1);

	//除了以上两种情况外 ， 数组名都指向数组中的首地址
	int * p = arr; // 直接对int*p进行赋值，也不会报错
	
	//数组名 是个指针常量 
	//arr = NULL;
    
	int len = sizeof(arr) / sizeof(int);
	printArray(arr, len);
    
	//数组下标是否可以为负数?
	p = p + 3;
	printf("p[-1] = %d\n", p[-1]);
	//上述代码等价于
	printf("p[-1] = %d\n",  *(p-1) );
}
```



结论： 一维数组数组名除了两种特殊情况外，可以理解为指向第一个元素的指针

特殊情况：

* sizeof数组名
* 对数组名取地址



### 5.2 数组指针定义方式

数组指针的定义方式：

1. 先定义出数组的类型，再定义数组指针变量
2. 先定义数组指针的类型，再定义数组指针变量
3. 直接定义数组指针变量



```C
//1、先定义出数组的类型，再定义数组指针变量
void test01()
{
	int arr[] = { 1, 2, 3, 4, 5 };

	typedef int(ARRAY_TYPE)[5]; //ARRAY_TYPE是一个数据类型，存放5个int元素的数组的类型
	ARRAY_TYPE * arrP = &arr; //arrP就是数组指针

	// *arrP 是 arr

	for (int i = 0; i < 5;i++)
	{
		//printf("%d\n", (*arrP)[i]);
		printf("%d\n", *((*arrP)+i)  );
	}
}

//2、先定义数组指针的类型，再定义数组指针变量
void test02()
{
	int arr[] = { 1, 2, 3, 4, 5 };

	typedef int(*ARRAY_TYPE)[5];

	// *arrP 是 arr
	ARRAY_TYPE arrP = &arr;

	for (int i = 0; i < 5; i++)
	{
		printf("%d\n", (*arrP)[i]);
		printf("%d\n", *((*arrP) + i));
	}
}

//3、直接定义数组指针变量
void test03()
{
	int arr[] = { 1, 2, 3, 4, 5 };

	int(*p)[5]  = &arr;

	//*p -> arr
	for (int i = 0; i < 5; i++)
	{
		printf("%d\n", (*p)[i]);
	}
}
```





### 5.3 二维数组与数组名

知道一维数组数组名的意义后，我们应该能够推到出二维数组数组名的意义了

```C
void test01()
{
	int arr[3][3] = {  
		{1,2,3},
		{4,5,6},
		{7,8,9}
	};

	//int arr2[3][3] = { 1, 2, 3, 4, 5, 6, 7, 8, 9 };
	//int arr3[][3] = { 1, 2, 3, 4, 5, 6, 7, 8, 9 };

	//二维数组名 等价与 一维数组的指针
	//除了对二维数组名 sizeof  或者  取地址以外，那么二维数组名 都是指向第一个一维数组的首地址

	int(*pArray)[3] = arr;

	//通过数组名 访问元素
	printf("arr[1][2] = %d\n", arr[1][2]);
	printf("arr[1][2] = %d\n", *(*(pArray + 1) + 2));
	printf("arr[1][2] = %d\n", *(*pArray + 5));
}


//二维数组做函数参数
//void printArray( int(*pArr)[3] ,int len1,int len2 )
//void printArray(int pArr[3][3], int len1, int len2)
void printArray(int pArr[][3], int len1, int len2)
{
	for (int i = 0; i < len1;i++)
	{
		for (int j = 0; j < len2;j++)
		{
			//printf("%d ", pArr[i][j]);

			printf("%d ",  *(*(pArr + i) + j) );
		}
		printf("\n");;
	}
}

void test02()
{
	int arr[3][3] = {
		{ 1, 2, 3 },
		{ 4, 5, 6 },
		{ 7, 8, 9 }
	};

	int row = sizeof(arr)/sizeof(arr[0]);
	int col = sizeof(arr[0]) / sizeof(int);
	printArray(arr, 3, 3);
}
```





## 6 结构体进阶

### 6.1 结构体基本使用

**结构体作用：**内置数据类型不够用的时候，我们可以利用结构体创建自定义的数据类型

下面案例中可以看出结构体的基本使用

```C
struct Person
{
	char name[64];
	int age; //不要给自定义数据类型赋初始值，因为只是在做定义 自定义数据类型
};
typedef struct Person myPerson;


typedef struct Person1
{
	char name[64];
	int age;
}myPerson;  //给类型起别名

void test01()
{
	myPerson p1 = {"aaa",10};
}


//在定义结构体的时候  顺便定义了一个结构体变量
struct Person2
{
	char name[64];
	int age;
}person2 = {"bbb",40};  //如果是变量 可以给初始化

void test02()
{
	person2.age = 20;
	strcpy(person2.name, "bbb");
}

struct
{
	char name[64];
	int age;
}person3 = {"ddd",100};  
//定义一个结构体变量 person3，后面无法使用这个类型，而这个类型只有一个变量

void test03()
{
	person3.age = 30;
	strcpy(person3.name, "ccc");
}

//结构体使用
struct Person
{
	char name[64];
	int age; //不要给自定义数据类型赋初始值，因为只是在做定义 自定义数据类型
};  
void test04()
{
	//在栈上使用
	struct Person p1 = { "aaa", 10 };
	printf("p1 姓名： %s  , 年龄:  %d \n", p1.name, p1.age);

	//在堆区使用
	struct Person * p2 = malloc(sizeof(struct Person));

	p2->age = 20;
	strcpy(p2->name, "bbb");
	printf("p2 姓名： %s  , 年龄:  %d \n", p2->name, p2->age);

	free(p2);
	p2 = NULL;

}
void printPerson( struct Person personArr[], int len)
{
	for (int i = 0; i < len;i++)
	{
		printf("姓名： %s  , 年龄:  %d \n", personArr[i].name, personArr[i].age);
	}
}
//结构体变量数组
void test05()
{
	//栈上分配内存
	struct Person persons[] =
	{
		{ "aaa", 10 },
		{ "bbb", 20 },
		{ "ccc", 30 },
		{ "ddd", 40 },
	};
	int len = sizeof(persons) / sizeof(struct Person);
	printPerson(persons, len);

	//堆区分配内存
	struct Person * personPoint = malloc(sizeof(struct Person) * 4);
	for (int i = 0; i < 4;i++)
	{
		sprintf(personPoint[i].name, "name_%d", i);
		personPoint[i].age = i + 10;
	}

	printPerson(personPoint, 4);

	free(personPoint);
	personPoint = NULL;
}
```



### 6.2 结构体赋值问题以及解决

这设计到一个深浅拷贝的问题

#### 6.2.1 结构体不包含指针的情况

```C
struct Person
{
	char name[64];
	int age;
};

void test01()
{
	struct Person p1 = { "Tom", 18 };

	struct Person p2 = { "Jerry", 20 };

	p1 = p2; //逐字节的进行拷贝

	printf("P1 姓名 ： %s, 年龄： %d\n", p1.name, p1.age);
	printf("P2 姓名 ： %s, 年龄： %d\n", p2.name, p2.age);
}

```



6.2.2 结构体包含指针的情况

```C
struct Person2
{
	char * name;
	int age;
};

void test02()
{
	struct Person2 p1;
	p1.name =  malloc(sizeof(char)* 64);
	strcpy(p1.name, "Tom");
	p1.age = 18;

	struct Person2 p2;
	p2.name = malloc(sizeof(char)* 128);
	strcpy(p2.name, "Jerry");
	p2.age = 20;

	//赋值
	//p1 = p2;

	//解决方式  手动进行赋值操作
	///////////////////////////////////
	if (p1.name!=NULL)
	{
		free(p1.name);
		p1.name = NULL;
	}

	p1.name = malloc(strlen(p2.name) + 1);
	strcpy(p1.name, p2.name);

	p1.age = p2.age;

	////////////////////////////////////

	printf("P1 姓名 ： %s  年龄： %d\n", p1.name, p1.age);
	printf("P2 姓名 ： %s, 年龄： %d\n", p2.name, p2.age);

	//堆区开辟的内容 自己管理释放
	if (p1.name != NULL)
	{
		free(p1.name);
		p1.name = NULL;
	}

	if (p2.name != NULL)
	{
		free(p2.name);
		p2.name = NULL;
	}

}
```

总结：当结构体中包含堆区的指针，要利用深拷贝解决浅拷贝的问题



### 6.3 结构体偏移量

意义：能够计算出结构体中属性相对于首地址的偏移量

案例：

```C
#include <stddef.h>
struct Teacher
{
	char a;
	int b;
};

void test01(){

	struct Teacher  t1;
	struct Teacher*p = &t1;


	int offsize1 = (int)&(p->b) - (int)p;  //通过地址查看成员b 相对于结构体 Teacher的偏移量
	int offsize2 = offsetof(struct Teacher, b);//通过宏函数查看

	printf("offsize1:%d \n", offsize1); //打印b属性对于首地址的偏移量
	printf("offsize2:%d \n", offsize2);
}

//通过偏移量 打印成员数据
void test02()
{
	struct Teacher t = { 'a', 10 };
	//打印c2的数据
	printf("t.b = %d\n", *(int *)((char*)&a + offsetof(struct Teacher, b)));

	printf("t.b = %d\n", *(int*)((int *)&a + 1 ));
}
//结构体嵌套结构体 计算偏移量方法
struct Teacher2
{
	char a;
	int  b;
	struct Teacher c;
};

void test03()
{
	struct Teacher2 t = { 'a', 10, 'b', 20 };

	int offset1 = offsetof(struct Teacher2, c);
	int offset2 = offsetof(struct Teacher, b);

	printf("%d\n", *(int *)(((char*)&t + offset1) + offset2));

	printf("%d\n", ((struct Teacher2 *)((char *)&t + offset1))->b);
}
```



### 6.4 内存对齐

#### 6.4.1 内存对齐意义

内存的最小单元是一个字节

CPU实际上将内存当成多个块，每次从内存中读取一个块，这个块的大小可能是2、4、8、16等，

我们来分析下非内存对齐和内存对齐的优缺点在哪？

![内存对齐](内存对齐.jpg)



如果没有对齐，为了访问一个变量可能产生二次访问。

有了内存对齐，可以提高操作系统访问内存的效率。



#### 6.4.2 结构体对齐计算规则

1.  第一个属性 从偏移量0位置开始存储
2. 第二个属性开始，放在 min(该类型的大小 ,对齐模数) 的整数倍上
3. 整体计算完毕后算总大小，结构体总大小必须是 min(该结构体中最大数据类型 ,对齐模数) 整数倍,不足要补齐



`#pragma pack(show) `可以查看对齐模数



练习：

```C
struct Student{
	int a;
	char b; 
	double c; 
	float d;
};

void test01()
{
	printf("sizeof = %d\n", sizeof(struct Student));
}

//2、结构体嵌套结构体， 按照子结构体最大的类型计算
struct Student2
{
	char a; 
	struct Student b; 
	double c; 
};

void test02()
{
	printf("sizeof = %d\n", sizeof(struct Student2));
}
```





## 7 文件读写

### 7.1 文件基本概念

#### 7.1.1 文件的基本概念

数据源的一种，最主要的作用是保存数据，如word、txt、头文件、源文件、exe等



#### 7.1.2 文件的分类

文件可以分为

* 磁盘文件
* 设备文件



**磁盘文件：** 

* 磁盘文件是计算机里的文件。存储信息不受断电的影响，存取速度相对于内存慢得多了

**设备文件：**

 * 操作系统中把每一个与主机相连的输入、输出设备看作是一个文件
 * 例如 显示器称为标准输出文件, 键盘称为标准输入文件



#### 7.1.3 磁盘文件的分类

计算机的存储在物理上是二进制的，所以物理上所有的磁盘文件本质上都是一样的：以字节为单位进行顺序存储。

从用户或操作系统的角度，将文件分为：

* 文本文件
* 二进制文件



**文本文件：**

* 基于字符编码，常见的编码有ASCII、UNICODE等
* 一般可以使用文本编辑器直接打开
* 如数字5678的存储形式 (ASCII码)为： 00110101 00110110 00110111 00111000



**二进制文件：**

* 基于值编码,自己根据具体应用,指定某个值是什么意思
* 把内存中的数据按其在内存中的存储形式原样输出到磁盘上
* 如数5678的存储形式(二进制码)为：00010110  00101110



### 7.2 文件指针

在C语言中，操作文件之前必须先打开文件；所谓“打开文件”，就是让程序和文件建立连接的过程。

打开文件之后，程序可以得到文件的相关信息，例如大小、类型、权限、创建者、更新时间等。

操作系统为了操作文件，提供了一堆文件的操作函数，而函数通过文件指针识别不同的文件

文件指针如下：

```C
typedef struct
{
	short           level;	//缓冲区"满"或者"空"的程度 
	unsigned        flags;	//文件状态标志 
	char            fd;		//文件描述符
	unsigned char   hold;	//如无缓冲区不读取字符
	short           bsize;	//缓冲区的大小
	unsigned char   *buffer;//数据缓冲区的位置 
	unsigned        ar;	 //指针，当前的指向 
	unsigned        istemp;	//临时文件，指示器
	short           token;	//用于有效性的检查 
}FILE;
```

FILE是系统使用typedef定义出来的有关文件信息的一种结构体类型

总结：如果我们想利用C语言操作一个文件，首先要获取到文件指针



### 7.3 文件打开与关闭

#### 7.3.1 fopen函数

函数原型： `FILE *fopen(char *filename, char *mode);`

功能：打开文件

参数：

​	filename - 需要打开的文件名，根据需要加上路径

​	mode - 打开文件的模式设置

返回值：

  	成功：文件指针

​	失败：NULL



| **打开模式** | **含义**                                                     |
| ------------ | ------------------------------------------------------------ |
| r或rb        | 以只读方式打开一个文本文件（不创建文件，若文件不存在则报错） |
| w或wb        | 以写方式打开文件(如果文件存在则清空文件，文件不存在则创建一个文件) |
| a或ab        | 以追加方式打开文件，在末尾添加内容，若文件不存在则创建文件   |
| r+或rb+      | 以可读、可写的方式打开文件(不创建新文件)                     |
| w+或wb+      | 以可读、可写的方式打开文件(如果文件存在则清空文件，文件不存在则创建一个文件) |
| a+或ab+      | 以添加方式打开文件，打开文件并在末尾更改文件,若文件不存在则创建文件 |

注：b是二进制模式的意思，b只是在Windows有效，在Linux用r和rb的结果是一样的



#### 7.3.2 fclose函数

函数原型：`int fclose(FILE *fp);`

* 打开的文件会占用内存资源，如果总是打开不关闭，会消耗很多内存
* 一个进程同时打开的文件数是有限制的，超过最大同时打开文件数，再次调用fopen打开文件会失败
* 如果没有明确的调用fclose关闭打开的文件，那么程序在退出的时候，操作系统会统一关闭



#### 7.3.3 文件打开关闭案例

```C
void test01()
{
	FILE *fp = NULL;
	fp = fopen("a.txt", "r"); 
	if (fp == NULL)
	{
		printf("打开失败\n");
		return;
	}

	printf("打开成功\n");

	fclose(fp);
}
```





### 7.4 文件读写

#### 7.4.1 按字符方式读写

**写文件 fputc**

​	函数原型： `int fputc(int ch, FILE *stream)`

**读文件 fgetc**

​	函数原型：`int fgetc(FILE * stream)`

**案例：**

```C
void test01()
{
	//打开文件
	FILE *fp = NULL;
	fp = fopen("a.txt", "w");
	if (fp == NULL)
	{
		printf("打开失败\n");
		return;
	}
	//操作文件
	char buf[] = "hello world\n";
	int i = 0;
	while (buf[i] != 0)
	{
		fputc(buf[i], fp);
		i++;
	}
	//关闭文件
	fclose(fp);
}
```



```C
void test02()
{
	//打开文件
	FILE *fp = NULL;
	fp = fopen("a.txt", "r");
	if (fp == NULL)
	{
		printf("打开失败\n");
		return;
	}

	//操作文件
	char ch = 0;
	while(  (ch = fgetc(fp)) != EOF  )
	{
		printf("ch = %c\n", ch);
	}

	//关闭文件
	fclose(fp);
}

```

注：==EOF==为文件结束标志，可以用来判断文件是否读取到文件尾





#### 7.4.2 按行方式读写

**写文件 fputs**

​	函数原型： `int fputs(const char *str, FILE *stream)`

**读文件 fgets**

​	函数原型：`int fgets(char *str, int size, FILE *stream)`

**案例：**



```C
void test01()
{
	char *buf[] = { 
        "床前明月光\n",
        "疑似地上霜\n",
        "举头望明月\n",
        "低头思故乡\n" 
    };

	FILE *fp = NULL;
	fp = fopen("a.txt", "w");
	if (fp == NULL)
	{
		printf("打开失败\n");
		return;
	}
	for (int i = 0; i < sizeof(buf) / sizeof(buf[0]); i++)
	{
		fputs(buf[i], fp);
	}

	fclose(fp);
}
```

```C
void test02()
{
	FILE *fp = NULL;
	fp = fopen("a.txt", "r");
	if (fp == NULL)
	{
		printf("打开失败\n");
		return;
	}

	char buf[1024] = { 0 };
	while (!feof(fp)){
		char temp[1024] = { 0 };
		fgets(buf, 1024, fp);
		if(feof(file_read))
		{
			break;
		}
		temp[strlen(buf)-1] = '\0';
		printf("%s", buf);
	}
	fclose(fp);
}
```

注：

* feof函数可以判断文件是否读取到了文件尾。如果是返回的值是1（真），否则为0（假）
* 使用 feof 常用的逻辑结构是先读在判断



#### 7.4.3 按格式化方式读写

**写文件 fprintf**

​	函数原型： `int fprintf(FILE * stream, const char * format, ...);`

**读文件 fscanf**

​	函数原型：`int fscanf(FILE * stream, const char * format, ...);`

**案例：**

```C
//格式化读写
struct Hero
{
	char name[16];
	int atk;
	int def;
};
void test01()
{
	struct Hero hero[5] = {
	{ "亚瑟",110, 200 },
	{ "赵云",150, 150 },
	{ "韩信",170, 130 },
	{ "蔡文姬",100, 180 },
	{ "斧头帮帮主",999, 999 }
	};

	FILE *fp = fopen("test1.txt", "w");
	if (fp == NULL)
	{
		printf("文件打开失败\n");
		return;
	}

	for (int i = 0; i < sizeof(hero) / sizeof(struct Hero); i++)
	{
		//1个中文两个字节
		int len = fprintf(fp, "%s %d %d\n", hero[i].name, hero[i].atk, hero[i].def); 
		printf("第%d行写入字节数：%d\n",i+1, len);//返回写入字符数
	}

	fclose(fp);

}
```

```C
void test02()
{
	struct Hero hero[5];
	memset(hero, 0, sizeof(hero));

	FILE *fp = fopen("test1.txt", "r");
	if (fp == NULL)
	{
		printf("文件打开失败\n");
		return;
	}
	int i = 0;
	while (!feof(fp))
	{
		fscanf(fp, "%s %d %d\n", hero[i].name, &hero[i].atk, &hero[i].def);
		i++;
	}
	int n = i;

	for (int i = 0; i < n; i++)
	{
		printf("name = %s atk = %d  def = %d\n", hero[i].name, hero[i].atk, hero[i].def);
	}

	fclose(fp);
}
```



#### 7.4.4 按块方式读写

**写文件 fprintf**

​	函数原型： `size_t fwrite(const void *ptr, size_t size, size_t nmemb, FILE *stream);;`

**读文件 fscanf**

​	函数原型： `size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream);`

**案例：**

```C
//块读写回顾
struct Hero{
	char name[64];
	int age;
};

void test01(){

	//写文件
	FILE* file_write = NULL;
	//写方式打开文件
	file_write = fopen("./test3.txt", "wb"); //wb 二进制方式写
	if (file_write == NULL){
		return;
	}

	struct Hero heros[4] = {
		{ "孙悟空", 33 },
		{ "韩信", 28 },
		{ "赵云", 45 },
		{ "亚瑟", 35 }
	};

	for (int i = 0; i < 4; i++){
		//参数1 数据地址  参数2  块大小   参数3  块数  参数4  文件指针
		fwrite(&teachers[i], sizeof(struct Hero), 1, file_write);
	}
	//关闭文件
	fclose(file_write);
}
```



```C
void test02()
{
	//读文件
	FILE* fp_read = NULL;
	fp_read = fopen("./test3.txt", "rb"); //二进制方式读
	if (fp_read == NULL){
		return;
	}

	struct Hero temps[4];
	fread(&temps, sizeof(struct Hero), 4, fp_read);
	for (int i = 0; i < 4; i++){
		printf("Name:%s\t Age:%d\n", temps[i].name, temps[i].age);
	}

	fclose(fp_read);
}
```



### 7.5 文件指针移动

文件指针移动这里主要介绍3个函数

* rewind


* fseek
* ftell      



#### 7.5.1 rewind

函数原型：`void rewind(FILE *stream );`

功能：把文件流（文件光标）的读写位置移动到文件开头



#### 7.5.2 fseek

函数原型： `int fseek(FILE *stream, long offset, int fromwhere);`

功能：移动文件流（文件光标）的读写位置。

fromwhere：取值如下：

​		SEEK_SET：从文件开头移动offset个字节

​		SEEK_CUR：从当前位置移动offset个字节

​		SEEK_END：从文件末尾移动offset个字节

fseek相关案例：

```C
struct Hero{
	char name[64];
	int age;
};

void test01(){

	//写文件
	FILE* file_write = NULL;
	//写方式打开文件
	file_write = fopen("./test4.txt", "wb"); //wb 二进制方式写
	if (file_write == NULL){
		return;
	}

	struct Hero heros[4] = {
		{ "孙悟空", 33 },
		{ "韩信", 28 },
		{ "赵云", 45 },
		{ "亚瑟", 35 }
	};

	for (int i = 0; i < 4; i++){
		//参数1 数据地址  参数2  块大小   参数3  块数  参数4  文件指针
		fwrite(&teachers[i], sizeof(struct Hero), 1, file_write);
	}
	//关闭文件
	fclose(file_write);
    
    
    //随机位置读取
    FILE* file_read = NULL;
	file_read = fopen("./test4.txt", "rb");
	if (file_read == NULL){
		return;
	}
	//创建临时的结构体 
	struct Hero temp;
	//读文件  移动到第三个结构体位置
	fseek(file_read, sizeof(Heros)* 2, SEEK_SET); //SEEK_SET从文件开始移动
	fread(&temp, sizeof(Heros), 1, file_read);
	printf("Name:%s Age:%d\n", temp.name, temp.age);

	memset(&temp, 0, sizeof(Heros));

	fseek(file_read, -(long)sizeof(Heros)* 2, SEEK_END); //SEEK_END从文件末尾移动
	fread(&temp, sizeof(Heros), 1, file_read);
	printf("Name:%s Age:%d\n", temp.name, temp.age);
    
	rewind(file_read); //把文件流（文件光标）的读写位置移动到文件开头，读第一个结构体
	fread(&temp, sizeof(Heros), 1, file_read);
	printf("Name:%s Age:%d\n", temp.name, temp.age);
    
	fclose(file_read);
}

```





#### 7.5.3 ftell

函数原型：`long ftell(FILE *stream);`

功能： 获取文件流（文件光标）的读写位置

ftell相关案例

```C
void test03()
{
	FILE *fp = fopen("test1.txt", "w"); 
	fputs("hello,斧头帮帮主", fp);
	fclose(fp);

	fp = fopen("test1.txt", "r");
	//将文件流指针定位到文件尾部
	fseek(fp, 0, SEEK_END);
	//得到文件流指针的偏移量
	int len = ftell(fp); 
	printf("len = %d\n", len); 

	fclose(fp);
}
```









### 7.6 文件相关案例

**解析配置文件案例**

案例需求：

* 游戏中有个配置文件，其中带`$`开头的代表注释信息，以`:`分隔的为正文内容
* `:` 左侧的起到索引作用，称为key值
* `:` 右侧的起到实值作用，称为value
* 将数据进行解析，并以键值对形式维护每组数据
* 创建一个数组，维护所有的键值对结构体
* 提供一个函数，可以通过key返回对应的value值



提示: 键值对的结构体设计可以如下

```C
struct ConfigInfo
{
	char key[64];
	char value[64];
};
```



配置文件config.txt 内容如下：

```c
$英雄的Id
heroId:1
$英雄的姓名
heroName:琦玉
$英雄的攻击力
heroAtk:99999
$英雄的防御力
heroDef:99999
$英雄的简介
heroInfo:无敌

```



## 8 函数指针

### 8.1 函数指针概念

函数指针：函数名称就是函数的入口地址，我们可以通过函数指针去指向函数的入口地址



```C
void func()
{
	printf("hello world\n");
}

int main() {

	printf("%p\n", func);
    
	system("pause");
	return EXIT_SUCCESS;
}
```



### 8.2 函数指针定义方式

函数指针定义方式有三种：

* 先定义函数类型，通过函数类型定义函数指针变量
* 先定义函数指针类型，再通过函数指针类型定义函数指针变量
* 直接定义函数指针变量



代码如下：

```C
void func(int a ,char b)
{
	printf("hello world\n");
}

void test01()
{
	//1、先定义函数类型，通过函数类型定义函数指针变量
	typedef void(FUNC_TYPE)(int,char);
	FUNC_TYPE * pFunc = func;
	pFunc(10,'a');


	//2、先定义函数指针类型，再通过函数指针类型定义函数指针变量
	typedef void(*FUNC_TYPE2) (int, char);
	FUNC_TYPE2 pFunc2 = func;
	pFunc2(10, 'a');

	//3、直接定义函数指针变量
	void(*pFunc3)(int, char) = func;
	pFunc3(10, 'a');

	//4、函数指针 和 指针函数区别
	//函数指针是指向函数的指针；
 	//指针函数是返回类型为指针的函数；
}

//函数指针的数组
void func1()
{
	printf("func1调用\n");
}
void func2()
{
	printf("func2调用\n");
}
void func3()
{
	printf("func3调用\n");
}

void test02()
{
	//函数指针数组
	void(*fun_array[3])();

	fun_array[0] = func1;
	fun_array[1] = func2;
	fun_array[2] = func3;

	for (int i = 0; i < 3;i++)
	{
		fun_array[i]();
	}
}
```



### 8.3 回调函数案例

当函数指针做函数参数的时候，利用函数指针调用所指的函数时，称为回调函数



案例1 ：提供一个函数，实现可以打印任何类型的元素

```C
void printText(void * data, void(*func)(void *))
{
	func(data);
}

void myPrintInt(void * data) //参数就是每个元素的地址
{
	int * num = data;
	printf("%d\n", *num);
}
void test01()
{
	int a = 10;
	printText(&a,myPrintInt);
}

struct Person
{
	char name[64];
	int age;
};
void myPrintPerson(void * data) //参数就是每个元素的地址
{
	struct Person * p = data;
	printf("姓名：%s 年龄： %d \n", p->name,p->age);
}
void test02()
{
	struct Person p = { "Tom", 100 };
	printText(&p, myPrintPerson);
}
```



案例2 ：提供一个函数，可以打印任意类型的数组

```C
void printALLArray(void *arr, int eleSize, int len , void(*myFunc)(void *) )
{
	char * p = arr;
	for (int i = 0; i < len;i++)
	{
		char * eleAddr = p + eleSize* i; //计算每个元素的地址
		myFunc(eleAddr);
	}
}

void myPrintInt(void * data) 
{
	int * num = data;
	printf("%d\n", *num);
}

struct Person
{
	char name[64];
	int age;
};

void myPrintPerson(void * data)
{
	struct Person * p = data;
	printf("姓名： %s  年龄： %d \n", p->name, p->age);
}

void test01()
{
	int arr[] = { 1, 2, 3, 4, 5 };

	int len = sizeof(arr) / sizeof(int);
	printALLArray(arr, sizeof(int), len, myPrintInt);

	struct Person personArr[] = 
	{
		{ "aaa", 10 },
		{ "bbb", 20 },
		{ "ccc", 30 },
		{ "ddd", 40 }
	};

	len = sizeof(personArr)/ sizeof(struct Person);
	printALLArray(personArr, sizeof(struct Person), len, myPrintPerson);
}
```



案例3 ：查找数组中的元素是否存在

```C
int findArrayEle(void *arr, int eleSize, int len, int(*myFunc)(void * , void *) , void * data)
{
	char * p = arr;
	for (int i = 0; i < len; i++)
	{
		char * eleAddr = p + eleSize* i; //计算每个元素的地址
		if (myFunc(eleAddr , data))
		{
			return 1;
		}
	}
	return 0;
}

int myFindPerson(void * data1, void * data2)
{
	struct Person * p1 = data1;
	struct Person * p2 = data2;
	if (strcmp(p1->name,p2->name) == 0  && p1->age == p2->age )
	{
		return 1;
	}
	return 0;
}

void test02()
{
	struct Person personArr[] =
	{
		{ "aaa", 10 },
		{ "bbb", 20 },
		{ "ccc", 30 },
		{ "ddd", 40 }
	};

	int len = sizeof(personArr) / sizeof(struct Person);

	struct Person p = { "aaa", 10 };
	int ret = findArrayEle(personArr, sizeof(struct Person), len, myFindPerson , &p);

	if (ret)
	{
		printf("找到了！\n");
	}
	else
	{
		printf("未找到!\n");
	}
}
```





## 9 预处理指令

### 9.1  预处理概念

* 预处理是在程序源代码被编译之前，由预处理器对程序源代码进行的处理。
* 这个阶段并不对程序的源代码语法进行解析，为下一步的编译做准备工作。



### 9.2 文件包含指令

* 文件包含是指一个源文件可以将另外一个文件的全部内容包含进来。
* Ｃ语言提供了#include命令用来实现“文件包含”的操作



**图示：**

![1590724863093](1590724863093.png)

**\#incude<>  #include""区别**

`< >` 表示包含系统头文件

`" "` 表示包含自定义头文件



### 9.3 宏定义

#### 9.3.1 宏常量

**语法：** `#define 名称  值`

**实例：** `#define MAX  1024`

```C
void test(){
	double r = 10.0;
	double s = PI * r * r;
	printf("s = %lf\n", s);
}
```



**总结：**

* 宏名一般用大写，以便于与变量区别
* 宏定义可以是常数(宏常量)、表达式（宏函数）
* 宏定义不作语法检查，只有在编译被宏展开后的源程序才会报错
*  宏名从定义到本源文件结束都可以使用
* 可以用#undef命令终止宏定义的作用域



#### 9.3.2 宏函数

**语法：** `#define 宏函数名(参数)  表达式` 

**实例：** `define SUM(x,y) (( x )+( y ))`

```C
#define SUM(x,y) (( x )+( y ))

void test(){
	int ret = SUM(10, 20);
	printf("ret:%d\n",ret);
}
```



**总结：**

* 在项目中，可以把一些短小而又频繁使用的函数写成宏函数

* 宏函数没有普通函数参数压栈、出栈上的开销，可以调高程序的效率

* 宏的名字中不能有空格，但是在替换的字符串中可以有空格

* 用括号括住每一个参数，并括住宏的整体定义。

* 用大写字母表示宏的函数名。

  ​



### 9.4 条件编译

* 一般情况下，源程序中所有的行都参加编译
* 但有时希望对部分源程序行只在满足一定条件时才编译，即对这部分源程序行指定编译条件

条件编译三种使用：

1. 测试存在   `#ifdef`
2. 测试不存在  `#ifndef`
3. 自定义条件 `#if`



示例：

`#ifdef`

```C
#define debug 
#ifdef debug
void func1()
{
	printf("Debug版本\n");
}
#else
void func1()
{
	printf("Release版本\n");
}
#endif 
```

`#ifndef`

```C
//防止头文件被重复包含引用；
#ifndef _SOMEFILE_H
#define _SOMEFILE_H

...

#endif
```

`#if`

```C
//做注释用
#if 0
    void myAdd()
    {
        printf("第一种写法\n");
    }
#else 
	void myAdd()
    {
        printf("第二种写法\n");
    }
#endif
```





### 9.5 特殊宏

```C
//	__FILE__			宏所在文件的源文件名 
//	__LINE__			宏所在行的行号
//	__DATE__			代码编译的日期
//	__TIME__			代码编译的时间

void test()
{
	printf("%s\n", __FILE__);
	printf("%d\n", __LINE__);
	printf("%s\n", __DATE__);
	printf("%s\n", __TIME__);
}
```





## 10 动态库的配置与使用

### 10.1 库的基本概念

* 库是已经写好的、成熟的、可复用的代码
* 每个程序都需要依赖很多底层库，不可能每个人的代码从零开始编写代码
* 我们的开发的应用中经常有一些公共代码是需要反复使用的，就把这些代码编译为库文件。
* 库可以简单看成一组目标文件的集合，将这些目标文件经过压缩打包之后形成的一个文件。



### 10.2 静态库的配置与使用

window下静态库配置步骤如下：

1. 创建新项目，编写库文件
2. 修改项目配置属性
3. 生成库文件
4. 测试并使用库


具体流程如下：



**1 创建项目**

* 创建一个空项目，项目名称例如：静态库
* 创建头文件和头文件，例如staticLib.h和staticLib.c

头文件添加如下代码：

```C
#pragma once

//加法运算，实现两个整型数字相加，并返回结果
int myadd(int a, int b);
```



源文件添加如下代码：

```C
#include "staticLib.h"

int myadd(int a, int b)
{
	return a + b;
}
```



**2 修改项目配置属性**

右键项目点击属性

在属性页面中选择  配置属性 - 常规 -配置类型 - 静态库 - 确定

![1590806174025](1590806174025.png)



**3 生成库文件**

此时，不需要运行程序，这里也没有写程序的入口

点击生成菜单，选择生成项目即可

![1590806388454](1590806388454.png)



成功后，在staticLib.c的==上层==的Debug目录中会有静态库.lib文件

**注：**

* 通常将生成的.lib文件和.h文件交给用户
* .h本身不需要，但是只有lib文件用户看不懂里面内容
* .h中写的库函数功能尽量描述清晰，起到说明作用





**4 测试并使用库**

创建新项目，导入生成的.lib和.h文件导入到项目中进行测试

![1590806974967](1590806974967.png)

运行查看结果，成功实现后，会打印出1 + 1 = 2 







### 10.3 动态库的配置与使用

#### 10.3.1 静态库优缺点

本章节主要讲的内容是动态库，主要是因为静态库缺点明显，利小于弊

优点：

* 静态库在程序的链接阶段被复制到了程序中


* 程序在运行时与库再无瓜葛，移植方便

缺点：

* 浪费空间和资源
  * 静态库所有内容都放在了程序中
  * 假设1个程序占用1MB，如果磁盘中有2000个程序，将近浪费了2GB空间
* 更新部署麻烦
  * 假设库文件为第三方厂商提供
  * 如果库文件有所改动，需要更新，那么整个程序需要重新编译发布给用户
  * 用户也需要重新安装整个程序



#### 10.3.2 动态库简介

为了解决静态库浪费空间和更新困难的两个问题，诞生了动态库

动态库链接思想：

* 将整个链接过程推迟到运行时候在进行
* 程序中用到了库函数，再从库中使用
* 更新时候，只需要替换库文件




#### 10.3.3  动态库配置和使用

window下动态库配置步骤如下：

1. 创建新项目，编写库文件
2. 修改项目配置属性
3. 生成库文件
4. 测试并使用库

具体流程如下：



**1 创建项目**

- 创建一个空项目，项目名称例如：动态库
- 创建头文件和头文件，例如dynamicLib.h和dynamicLib.c

头文件添加如下代码：

```c
#pragma once

//减法运算，实现两个整型数字相减，并返回结果
__declspec(dllexport) int mysub(int a, int b);
```

注：

* `__declspec(dllexport)` 为导出函数，只有导出函数才可以被外部程序使用



源文件添加如下代码：

```c
#include "dynamicLib.h"

int mysub(int a, int b)
{
	return a - b;
}
```





**2 修改项目配置属性**

右键项目点击属性

在属性页面中选择  配置属性 - 常规 -配置类型 - 动态库 - 确定

![1590809778154](1590809778154.png)



**3 生成库文件**

不需要运行程序，点击生成菜单，选择生成项目即可

成功后，在staticLib.c的==上层==的Debug目录中会有核心文件：动态库.lib 文件 以及 动态库.dll文件

**注：**

- 通常将生成的.dll文件 .lib文件和.h文件交给用户
- .h中写的库函数功能尽量描述清晰，起到说明作用
- 本案例演示库程序为中文，实际开发中最好用英文
- 动态库中的lib文件中存放导出函数的声明，具体实现在dll中，这与静态库中的lib是不同的



**4 测试并使用库**

创建新项目，导入生成的.dll 、 .lib和 .h文件导入到项目中进行测试

![1590810554561](1590810554561.png)

运行查看结果，成功实现后，会打印出1 - 1 = -1

注： 如果不把文件导入项目中，也可以采用代码中添加 `#pragma comment(lib,"./动态库.lib")` 使用动态库





## 11 递归函数

### 11.1 普通函数调用

学习递归函数前，我们先要搞清楚普通函数的调用流程

```C
#define _CRT_SECURE_NO_WARNINGS
#include<stdio.h>
#include<string.h>
#include<stdlib.h>

void funB(int a) 
{
	printf("funB中的 a = %d\n", a);
}

void funA(int a) 
{
	funB(a - 1);
	printf("funA中的 a = %d\n", a);
}

int main() {

	funA(2);
	printf("main\n");

	system("pause");
	return EXIT_SUCCESS;
}
```

思考上面代码运行的结果，理解代码的执行流程

**流程分析图：**



![1590812031315](1590812031315.png)

执行结果为：

```C
funcB中的 a = 1;
funcA中的 a = 2;
main
请按任意键继续...
```





### 11.2 递归函数调用

如果11.1已经看懂，那么接下来分析以下代码的运行结果：

```C
#define _CRT_SECURE_NO_WARNINGS
#include<stdio.h>
#include<string.h>
#include<stdlib.h>

void funA(int a) 
{
	if (a == 1)
	{
		printf("funA中的 a = %d\n", a);
		return;
	}

	funA(a - 1);
	printf("funA中的 a = %d\n", a);
}

int main() {

	funA(2);
	printf("main\n");

	system("pause");
	return EXIT_SUCCESS;
}
```

**流程分析图：**

![1590812756535](1590812756535.png)



执行结果为：

```c
funcA中的 a = 1;
funcA中的 a = 2;
main
请按任意键继续...
```

上面这种，在函数中调用自身，就属于递归函数

注：

* 递归函数必须有退出条件，否则出现无限递归



### 11.3 递归函数案例

案例1：逆序遍历字符串

```C
void reversePrint(char * p)
{
	if (*p == '\0')
	{
		return;
	}

	reversePrint(p + 1);
	printf("%c", *p);
}

void test01()
{
	char * str = "abcdefgh";
	reversePrint(str);
}
```



案例2：获取斐波那契数列指定位置元素

斐波那契数列：1、1、2、3、5、8...

```C
int fibonacci(int pos)   
{
	if (pos == 1 || pos == 2)
	{
		return 1;
	}

	return fibonacci(pos - 1) + fibonacci(pos - 2);
}

void test02()
{
	int num = fibonacci(10);
	printf("斐波那契数列中第十个元素为：%d\n", num);
}
```


