### C 语言入门

### 1. 第一个 C 程序
- 创建`hello.c`文件；
- 使用`gcc hello.c -o hello`编译该文件；
- 运行`./hello`；

```c
#include <stdio.h>

int main()
{
	printf("Hello World!\n");
	return 0;
}
```

### 2. 变量和常量
- `change.c`
- `gcc change.c -o change`

```c
#include <stdio.h>

int main()
{
	int price = 0;
	printf("请输入金额（元）：");
	scanf("%d", &price);

	int change = 100 - price;

	printf("找您 %d 元\n", change);
}
```

- 变量定义的一般形式：
  - `<类型名称> <变量名称>;`
- 变量初始化：
  - `<类型名称> <变量名称> = <初始值>;`
  - `int price = 0;`

### 3. 表达式
- 表达式，是由一系列**运算符**和**算子**的组合，用来计算一个值。

### 4. 控制语句
#### 4.1 `if...else...`
#### 4.2 循环
- `while`
- `do...while...`
- `for`

```c
// 1，猜数：do...while...

#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main()
{
	srand(time(0));
	int number = rand() % 100 + 1;
	int count = 0;
	int a = 0;
	printf("我已经想好了一个 1 到 100 之间的数。");
	// do... while... 循环
	do {
		printf("尝试猜一下1 到 100 之间的数：\n");
		scanf("%d", &a);
		count++;
		if (a > number) {
			printf("你猜的数大了。\n");
		} else if (a < number) {
			printf("你猜的数小了。\n");
		}
	} while (a != number);

	printf("恭喜，你用了 %d 次就猜到了答案。\n", count);

	return 0;
}

```

- `while`循环
```c
#include <stdio.h>

/**
 * 最大公约数 -- 辗转相除法
 *  1，如果 b 等于 0，计算结束，a 就是最大公约数
 *  2，否则，计算 a 除以 b 的余数，让 a 等于 b，而 b 等于那个余数；
 *  3，回到第一步
 * 
 * a	 b	 t
 * 12   18   12
 * 18   12   6
 * 12    6   0
 * 6     0
 */

int main()
{
	int a, b;
	int m;
	scanf("%d %d", &a, &b);

	while (b != 0) {
		m = a % b;
		a = b;
		b = m;
	}
	printf("最大公约数：%d\n", a);
}
```

### 5. 数据类型
- 整数
  - `char`、`short`、`int`、`long`、`long long`
  - `int` 是用来表达寄存器的大小；
- 浮点数
  - `float`、`double`、`long double`
- 逻辑
  - `bool`
- 指针
- 自定义类型

- `sizeof` 运算符
  - 可以给出某个类型或变量在内存中所占据的字节数。
  - `size_demo.c`
```c
#include <stdio.h>

int main()
{
	printf("sizeof(char)=%ld\n", sizeof(char));
	printf("sizeof(short)=%ld\n", sizeof(short));
	printf("sizeof(int)=%ld\n", sizeof(int));
	printf("sizeof(long)=%ld\n", sizeof(long));
	printf("sizeof(long long)=%ld\n", sizeof(long long));
	return 0;
}
```

### 6. 函数
- 函数是一块代码，接收零个或多个参数，做一件事情，并返回零个或一个值；
- 函数原型
- 参数传递
- 变量的生命周期和作用域

### 7. 数组
- 定义数组：`<类型> 变量名称[元素数量];`
- `num_count.c`

```c
// 输入数量不确定的 [0,9] 范围内的整数，统计每一种数字出现的次数
// 输入 -1 表示结束

#include <stdio.h>

int main()
{
	const int number = 10;
	int x;
	int count[number];
	// 初始化数组
	for (int i = 0; i < number; i++) {
		count[i] = 0;
	}
	scanf("%d", &x);
	while (x != -1) {
		if (x >= 0 && x <= 9) {
			count[x]++;
		}
		scanf("%d", &x);
	}
	// 打印结果
	for (int i = 0; i < 10; i++) {
		printf("%d: %d\n", i, count[i]);
	}
	return 0;
}
```

- 判断输入的数字，是否存在于数组中
- `search_array.c`
```c
#include <stdio.h>

// 函数原型
int search(int key, int a[], int length);

int main()
{
	// 初始化数组
	int a[] = {2, 4, 6, 7, 1, 3, 5, 9, 11, 23, 15, 47};

	int x;
	int loc;
	printf("请输入一个数字：");
	scanf("%d", &x);

	// 计算数组长度：sizeof(a)/sizeof(a[0])
	loc = search(x, a, sizeof(a)/sizeof(a[0]));
	if (loc != -1) {
		printf("%d 在第 %d 个位置上\n", x, loc);
	} else {
		printf("数字 %d 不存在\n", x);
	}

	return 0;
}

// 数组作为函数参数时，往往必须再用另一个参数来传入数组的大小
int search(int key, int a[], int length)
{
	int ret = -1;
	for (int i = 0; i < length; i++) {
		if (a[i] == key) {
			ret = i;
			break;
		}
	}
	return ret;
}
```

#### 7.1 数组的赋值
- 不能使用一个数组变量本身初始化另外一个数组；
- 如果要把一个数组的所有元素交给另外一个数组，必须采用遍历；

```c
int a[] = {2,3,6,11,45};
// 用以下方法，初始化 b[] 是错误的
// error: array initializer must be an initializer list or wide string literal
int b[] = a;

// 正确初始化方法
for (int i = 0; i < length; i++) {
	b[i] = a[i];
}

// array_init.c
#include <stdio.h>

int main()
{
	int a[] = {13, 4,45, 56, 57, 34};
	char b[] = "hello world";
	int b_len = sizeof(b)/sizeof(b[0]);
	for (int i = 0; i < b_len; i++) {
		printf("b[%d] = %c\n", i, b[i]);
	}
}
```

### 8. 指针
#### 8.1 取地址运算符 `&`
```c
// 读取数据
scanf("%d", &i);

// 打印地址
printf("%p\n", &i);
```

- 打印数组地址
```c
#include <stdio.h>

int main()
{
	int a[10];

	printf("&a: %p\n", &a);
	printf("a: %p\n", a);
	printf("&a[0]: %p\n", &a[0]);
	printf("&a[1]: %p\n", &a[1]);
}
```

#### 8.2 指针
- 指针就是保存地址的变量。
  - `int *p = &i`;
- 普通变量的值是实际的值；
- 指针变量的值是具有实际值的变量的地址；

![指针变量](https://github.com/xianliu18/ARTS/tree/master/C%26C%2B%2B/images/指针变量.png)

- 单目运算符`*`：用来访问指针的值所表示的地址上的变量；
- `swap_demo.c`

```c
#include <stdio.h>

void swap(int *pa, int *pb);

int main()
{
	int a = 22;
	int b = 33;
	swap(&a, &b);
	printf("a = %d, b = %d\n", a, b);

	return 0;
}

void swap(int *pa, int *pb)
{
	int t = *pa;
	*pa = *pb;
	*pb = t;
}
```

#### 8.3 指针应用场景
- 场景一：函数返回多个值，某些值就只能通过指针返回；
  - 传入的参数实际上是需要保存带回的结果的变量；
```c
#include <stdio.h>

void minmax(int a[], int len, int *min, int *max);

int main()
{
	int a[] = {3, 56, 67, 88, 23, 5445, 55, 199};
	int min, max;
	// 数组的大小
	printf("main sizeof(a)=%lu\n", sizeof(a));
	minmax(a, sizeof(a)/sizeof(a[0]), &min, &max);
	printf("min = %d, max = %d\n", min, max);

	return 0;
}

// 需要返回两个值：最小值和最大值
void minmax(int a[], int len, int *min, int *max)
{
	int i;
	*min = *max = a[0];

	// 数组作为参数时，数组的大小
	// sizeof on array function parameter will return size of 'int *' instead of 'int []'
	printf("minmax sizeof(a)=%lu\n", sizeof(a));
	for (i = 1; i < len; i++) {
		if (a[i] < *min) {
			*min = a[i];
		}
		if (a[i] > *max) {
			*max = a[i];
		}
	}
}
```

- 场景二：函数返回运算的状态，结果通过指针返回

```c
#include <stdio.h>

/**
 * @return 如果除法成功，返回 1；否则返回 0 
 */
int divide(int a, int b, int *result);

int main()
{
	int a = 5;
	int b = 2;
	int c;
	if (divide(a, b, &c)) {
		printf("%d/%d = %d\n", a, b, c);
	}
	return 0;
}

int divide(int a, int b, int *result)
{
	int ret = 1;
	if (b == 0){
		ret = 0;
	} else {
		*result = a/b;
	}
	return ret;
}
```

- 常见错误：
  - 定义了指针变量，还没有指向任何变量，就开始使用指针。
  - `pointer_init.c`
```c
#include <stdio.h>

int main()
{
	int a = 11;
	int b = 44;

	// segmentation fault
	// int *p; 或 int *p = 0;
	int *p = 0;
	*p = 55;
	printf("*p = %d\n", *p);
	return 0;
}
```

#### 8.4 数组变量是特殊的指针
- 函数参数表中的数组实际上是指针。
- 数组变量本身就表达的是地址，
  - `int a[10]; int *p = a;`：不需要使用 `&a` 取地址；
  - 但是数组的元素表达的是变量，需要用 `&`取地址
- 数组变量是 const 的指针，所以不能被**赋值**；
  - `int a[]` 等同于 `int * const a`

#### 8.5 指针与 const
![指针与 const](https://github.com/xianliu18/ARTS/tree/master/C%26C%2B%2B/images/指针与const.png)

- 指针是 `const`:
  - 表示一旦得到了某个变量的地址，不能再指向其他变量。

```c
int i = 100;
int * const q = &i;	// 指针 q 是 const
*q = 99;	// OK
q++;		// ERROR
```

- 值是 `const`
  - 表示不能通过这个指针去修改那个变量（并不能使得那个变量成为 const）

```c
int i = 100;
int j = 50;
const int *p = &i; // 或 int const* p2 = &i;
*p = 88;	// ERROR  (*p) 是 const

i = 26;		// OK
p = &j;		// OK
```

- const 数组
  - `const int a[] = {1, 2, 3, 4, 5};`
  - 数组变量已经是 `const` 的指针了，这里的 `const` 表明数组的每个单元都是 `const`;
  - 必须通过**初始化**进行赋值；
- `const` 数组使用场景：
  - 把数组传入函数时，传递的是地址，所以那个函数内部可以修改数组的值；
  - 为了保护数组不被函数破坏，可以设置参数为`const`
    - `int sum(const int a[], int len);`

#### 8.6 指针运算
```c
#include <stdio.h>

int main()
{
	char ac[] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
	char *p = ac;
	printf("p = %p\n", p);
	// 指针运算
	printf("p+1 = %p\n", p+1);	// 增加的大小为 sizeof(char)

	// *(p+n) 等价于 a[n]

	// 数组类型换为 int
	int num[] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
	int *q = num;
	printf("q = %p\n", q);
	printf("q+1 = %p\n", q+1);	// 增加的大小为 sizeof(int)

	// 两个指针相减
	int *m = &num[5];
	printf("diff: %d\n", m - q);

	return 0;
}
```

### 9. 字符串
- 以`0`(整数 0)结尾的一串字符
  - `0`或`\0`是一样的，但是和`'0'`不同；
- `0`标志字符串的结束，但它不是字符串的一部分
  - 计算字符串长度的时候，不包含这个 `0`；
- 字符串以**数组**的形式存在，以数组或指针的形式访问；
  - 更多的是以**指针**的形式访问；

```c
// 字符串变量
char *str = "Hello";
char word[] = "Hello";
char line[10] = "Hello";
```

- C 语言的字符串是以字符数组的形态存在
  - 不能用运算符对字符串做运算；
  - 通过数组的方式可以遍历字符串；
- 唯一特殊的地方是字符串字面量可以用来初始化字符数组；

```c
#include <stdio.h>

int main()
{
	int i = 0;
	// 字符串位于程序的代码段，只读
	char *str = "Hello World";
	char *str2 = "Hello World";
	
	printf("i 地址：%p\n", &i);

	printf("s = %p\n", str);
	printf("s2 = %p\n", str2);

	// 由于 Hello World 放在只读代码段，所以此处程序会崩溃
	// str[0] = 'b';
	// printf("%s\n", str);

	// 如果需要修改字符串，应该使用数组
	char str3[] = "Hello World";
	printf("s3 = %p\n", str3);
	str3[0] = 'M';
	printf("s3 = %s\n", str3);

	return 0;
}
```

#### 9.1 字符串输入输出
- `scanf` 读入一个单词（读到空格、tab或回车为止）；
- `scanf` 是不安全的，因为不知道要读入的内容的长度；

```c
#include <stdio.h>

int main()
{
	char word[8];
	char word2[8];
	char word3[8];

	scanf("%s", word);
	scanf("%s", word2);

	// 7 表示最多只能读取 7 个字符
	scanf("%7s", word3);
	printf("%s##%s##%s##\n", word, word2, word3);

	return 0;
}
```

#### 9.2 常见错误
- 以为`char *`是字符串类型，定义了一个字符串类型的变量 `str` 就可以直接使用了；
  - 由于没有对 `str` 初始化为 0，所以不一定每次运行都出错；
```c
char *str;
scanf("%s", str);
```

### 10. 枚举
- 枚举是一种用户定义的数据类型，关键字`enum`
  - `enum 枚举类型名字{名字 0，... 名字 n};`

### 11. 结构
```c
#include <stdio.h>

// 声明结构类型
struct date {
	int month;
	int day;
	int year;
};

int main(int argc, char const *argv[])
{
	struct date today;

	today.month = 12;
	today.day = 26;
	today.year = 2022;

	printf("Today's date is %i-%i-%i\n", today.year, today.month, today.day);

	struct date day2;

	day2 = (struct date){8, 17, 2022};

	struct date day3;
	// 结构赋值
	day3 = day2;

	printf("day2's date is %i-%i-%i\n", day2.year, day2.month, day2.day);
	printf("day3's date is %i-%i-%i\n", day3.year, day3.month, day3.day);

	return 0;
}
```

#### 11.1 指向结构的指针
```c
struct date {
	int month;
	int day;
	int year;
} myday;

struct date *p = &myday;

(*p).month = 12;

// 等同于,  -> 表示指针所指的结构变量中的成员
p->month = 12;
```

#### 11.2 类型定义
- `typedef` 用来声明一个已有的数据类型的新名字：
  - `typedef int Length;` 使得 `Length` 成为 `int` 类型的别名；
```c
// 自定义类型 Date
typedef struct {
	int month;
	int day;
	int year;
} Date;

Date d = {9, 1, 2022};
```

#### 11.3 联合(union)
- 所有的成员共享一个空间；
- 同一时间只有一个成员是有效的；
- union 的大小是其最大的成员；

### 12. 全局变量
- 定义在函数外面的变量是全局变量；
- 全局变量具有全局的生存期和作用域：
  - 它们与任何函数都无关；
  - 在任何函数内部都可以使用它们；

#### 12.1 静态本地变量
- 在本地变量定义时，加上`static`修饰符就成为静态本地变量；
- 当函数离开的时候，静态本地变量会继续存在并保持其值；
- 静态本地变量的初始化只会在第一次进入这个函数时做，以后进入函数时会保持上次离开时的值；
- `static_demo.c`
```c
#include <stdio.h>

int f(void);

int main(int argc, char const *argv[])
{
	f();
	f();
	f();
	return 0;
}

int f(void)
{
	static int all = 1;
	printf("in %s all=%d\n", __func__, all);
	all += 2;
	printf("agn in %s all=%d\n", __func__, all);
	return all;
}
```

- 静态本地变量实际上是特殊的全局变量；
- 它们位于相同的内存区域；
- 静态本地变量具有全局的生命周期，函数内的局部作用域；
  - `static`在这里的意思是局部作用域(本地可访问)；
- `static_mem.c`
```c
#include <stdio.h>

int f(void);

// 全局变量
int gAll = 12;

int main(int argc, char const *argv[])
{
	f();
	return 0;
}

int f(void)
{
	int a = 66;
	static int all = 1;
	// 本地变量和全局变量位于相同的内存区域
	printf("&gAll=%p, &all=%p\n", &gAll, &all);

	// 普通本地变量的内存区域
	printf("&a=%p, &b=%p\n", &a, &b);
	return all;
}
```

#### 12.1 `*`返回指针的函数
- 返回本地变量的地址是危险的；
- 返回全局变量或静态本地变量的地址是安全的；
- 返回在函数内`malloc`的内存是安全的，但是容易造成问题；
- 最好的做法是**返回传入的指针**；
- `local_return.c`
```c
#include <stdio.h>

int *f(void);
void g(void);

int main(int argc, char const *argv[])
{
	int *p = f();
	printf("*p=%d\n", *p);
	g();
	printf("*p=%d\n", *p);
}

int *f(void)
{
	int i = 12;
	return &i;
}

void g(void)
{
	int k = 24;
	printf("k=%d\n", k);
}
```

#### 12.2 宏定义
- 以`#`开头的是编译预处理指令；
- `#define`用来定义一个宏；
  - `#define <名字> <值>`
  - 注意**没有结尾的分号**，因为不是 C 的语句；
  - 名字必须是一个单词，值可以是各种东西；
  - 在 C 语言的编译器开始编译之前，编译预处理程序(cpp)会把程序中的名字换成值，完全的文本替换；
  - `gcc --save-temps`：可以保留编译预处理后的文件；
```c
#include <stdio.h>

//const double PI = 3.14159;
// 定义宏
#define PI 3.14159

int main()
{
	printf("%f\n", 2*PI*3.0);
	return 0;
}
```

#### 12.3 头文件
- `#include` 有两种形式来指出要插入的文件 `""`还是`<>`
	- `""`要求编译器首先在当前目录(`.c`文件所在的目录)寻找这个文件，如果没有，再到编译器指定的目录去找；
	- `<>`让编译器只在指定的目录去找；
- 在**使用和定义**这个函数的地方都应该`#include`这个头文件；
- 一般做法就是任何`.c`都有对应的同名`.h`，把所有对外公开的**函数原型和全局变量**的声明都放进去；
- 不对外公开的函数（`static`）：
  - 在函数前面加上`static`就使得它成为只能在所在的编译单元中被使用的函数；
  - 在全局变量前面加上`static`就使得它成为只能在所在的编译单元中被使用的全局变量；

```c
int i;			// 变量的定义
extern int i;	// 变量的声明
```

#### 12.4 格式化的输入输出
- `printf`
  - `%[flags][width][.prec][hlL]type`
- `scanf`
  - `%[flags]type`

<br/>

**参考资料：**
- [C 语言程序设计（翁恺）](https://www.bilibili.com/video/BV1XZ4y1S7e1)