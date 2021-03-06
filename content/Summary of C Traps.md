Title: Summary of C Traps
Date: 2015/12/8
Category: Programming Language
Tags: C
Author: WangXinyu
# C陷阱总结
####版权声明：本文为作者原创，欢迎批评指正，谢绝转载。

本文记录笔者在学习和实践中遇到的C问题，将随着学习的深入不断更新。
## 1. const修饰符
const指常量、常数。const与#define相比，效率更高。

	const int i;				/* i不可变 */
	int const i;				/* i不可变 */
	int const a[3]={1, 2, 3};	/* 数组不可变 */
	const int a[3]={1, 2, 3};	/* 数组不可变 */
	const int *p;				/* p 可变，p 指向的对象不可变 */
	int const *p;				/* p 可变，p 指向的对象不可变 */
	int *const p;				/* p 不可变，p 指向的对象可变 */
	const int *const p;			/* 指针p 和p 指向的对象都不可变 */

## 2. volatile修饰符
volatile指易变的，比如端口数据等变量应当使用volatile修饰。volatile告诉编译器，这个变量是反复无常的，编译器每次用这个变量时，必须从内存中取它的值，编译器对访问该变量的代码不再进行优化。
## 3. 可变参数"..."
"..."表示参数的个数和类型是可变的，printf()就用到了可变参数，它的定义如下：

int printf( const char* format, ... );

在函数内部通过前面固定参数的地址获取可变参数的地址，从而可以在函数内部处理这些可变参数。
## 4. 语句块
由{}括起来的若干条语句组成一个语句块。如下所示，语句块构成一个作用域，函数内的i和语句块中的i不同，语句块中的j的作用域只是语句块内。

	void foo(void)	{		int i = 0;		{			int i = 1;			int j = 2;
			printf("i=%d, j=%d\n", i, j);		}		printf("i=%d\n", i); /* cannot access j here */	｝
## 5. 全局变量和局部变量的初始化
局部变量可以用任意类型相符的表达式来初始化，而全局变量只能用常量表达式初始化。全局变量的初始值要求保存在编译生成的目标代码中，所以必须在编译时就能计算出来。

如果全局变量在定义时不初始化，则初始值是0，也就是说，整型的就是0，字符型的就是'\0'，浮点型的就是0.0。如果局部变量在定义时不初始化，则初始值是不确定的，所以，局部变量在使用前一定要先赋值，不管是通过初始化还是赋值运算符。

## 6. 结构体的初始化和赋值
如果Initializer中的数据比结构体的成员多，编译器会报错，但如果只是末尾多个逗号不算错。如果Initializer中的数据比结构体的成员少，未指定的成员将用0来初始化，就像未初始化的全局变量一样。例如以下几种形式的初始化都是合法的，但结构体直接赋值是非法的。

	struct complex_struct { double x, y; };
	struct complex_struct z1 = { x, 4.0, };	/* z1.x=3.0, z1.y=4.0 */
	struct complex_struct z2 = { 3.0, };		/* z2.x=3.0, z2.y=0.0 */
	struct complex_struct z3 = { };				/* z3.x=0.0, z3.y=0.0 */
	
	struct complex_struct z1;
	z1 = { 3.0, 4.0 };							/* illegal */
	
## 7. #include的尖括号<>和引号""
对于用尖括号<>包含的头文件，gcc首先查找-I选项指定的目录，然后查找系统的头文件目录（Linux通常是/usr/include）。

而对于用引号包含的头文件，gcc首先查找包含头文件的.c文件所在的目录，然后查找-I选项指定的目录，然后查找系统的头文件目录。
## 8. 指针数组和指向数组的指针
int \*a[10]是指针数组，而int (\*a)[10]是指向数组的指针。我们可以认为[]比\*有更高的优先级，如果a先和*结合则表示a是一个指针，如果a先和[]结合则表示a是一个数组。

	/* int *a[10];可以拆成以下两句 */
	typedef int *t;	t a[10];

	/* int (*a)[10];可以拆成以下两句 */	typedef int t[10];	t *a;
	
持续更新中......
