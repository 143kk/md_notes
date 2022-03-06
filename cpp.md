

## Initialization

**初始化和赋值是两个不同的操作**

```cpp
int a = 1;
int b(1);
int c = {1}; // C++11
int d{1} // C++11 可以省略等号
int x{} // 变量被初始化为0
```

**如果不对函数内部定义的变量进行初始化，变量的值将是创建前相应内存单元保存的值。总之，变量的值是不确定的。**

## Main

```cpp
int main(int argc, char * argv[])
{
    for (int i = 0; i < argc; i++)
        cout << i << ": " << argv[i] << endl;
}
```

`argc`为参数个数, `argv[0]`为文件路径, `argv[1...argc]`为参数。

## Data types

c++对大小写敏感

### 各种后缀

```
2 // int
2.0 // double
2l // long
2.0l // long double

2f // Invalid
2.0f // float
2.f // float

2u // unsigned int
2ul // unsigned long
2lu // unsigned long
2ll, 2LL// long long


1e9 // double
1e9f // float
```



----

### Integer types

c++并没有明确规定每个类型的width in bits.

如果要固定width，使用`int8_t` `int16_t`... (Fixed width integer types)

以下结果测试于`g++ (Ubuntu 9.3.0-17ubuntu1~20.04) 9.3.0`环境下

- `char` 1 bytes
- `short` 2 bytes 
- `int` 4 bytes 
- `long` 8 bytes
- `long long`(C++11) 8 bytes

long和long long 

```cpp
#include <iostream>
#include <climits>

using namespace std;

int main() {
	int n_int = INT_MAX;
	short n_short = SHRT_MAX;
	long n_long = LONG_MAX;
	long long n_llong = LLONG_MAX;
	cout << n_int << endl << n_short << endl << n_long << endl << n_llong << endl;
	return 0;
}

/* g++ (Ubuntu 9.3.0-17ubuntu1~20.04) 9.3.0
2147483647
32767
9223372036854775807
9223372036854775807
*/
```

`sizeof`不仅可以对变量使用，还可以对类型使用，但类型名要放到括号中，如`sizeof(int)`

`unsigned` 是`unsigned int`的缩写

#### Overflow 

```cpp
int a = 56789;
int b = 56789;
int c = a*b;
cout << c << endl;
// output: -1069976775
// correct result: 3224990521
```

$3224990521_{10}$的二进制有32位，第三十二位为1。3224990521直接被赋值给了c，符号位为1，所以输出是-1069976775。(-1069976775的补码和3224990521的二进制相同)

#### Integral promotion

> In particular, arithmetic operators do not accept types **smaller than int** as arguments.

```cpp
unsigned char a = 255;
unsigned char b = 1;
int c = a + b; // c = 256
```

https://en.cppreference.com/w/cpp/language/implicit_conversion

### size_t

`size_t`的width取决于平台，如在32位系统上，size_t有4个字节；64位系统上有8个字节。

同时，size_t是无符号的，是`sizeof`的返回类型

### Character

- char
- wchar_t
- char16_t
- char_32_t

c++并没有规定char有无符号，`char`有无符号取决于编译器和平台

在本机环境下，`char`实际上是`signed char`

```cpp
#include <iostream>
#include <climits>

using namespace std;

int main(int argc, char *argv[]) {
        cout << "CHAR_MAX: " << CHAR_MAX << endl;
        cout << "CHAR_MIN: " << CHAR_MIN << endl;

        return 0;
}

// output
// CHAR_MAX: 127
// CHAR_MIN: -128
```

`\b`代表退格，会使光标向前移动一位。

```cpp
#include <iostream>

using namespace std;

int main() {
	int code;
	cout << "Please enter your code: ______\b\b\b\b\b\b";
	cin >> code;
	cout << code << endl;
	return 0;
}
```

中文

```cpp
char16_t ch1 = u'张';
char32_t ch2 = U'张';
```

### Bool

`bool`被转化为整型时，`true`被转化为1, false被转化为0

隐式转换，非0值转换为true,0转换为false

```cpp
bool a = true;
cout << a;

//output: 1
```



### 浮点型

- float
- double
- long double

E表示法的数字是以浮点格式存储，如`8.92E+4`, `22.33E-12`

浮点数字面值属于double类型，如果希望为float类型，要加后缀f或F，如`8.92E+4f`。

相似的，`long double`加l或L后缀

可以从`cfloat`头文件中找到关于浮点数的限制

### 数组

`typeName arrayName[arraySize]`，其中`arraySize`必须为字面值或常量，不能是变量。

初始化数组并赋值：`int a[4] = {1, 2, 3, 4}`

`sizeof arrayName`得到的是整个数组的字节数

**只有定义数组时才能初始化**

```
int a[4];
a = {1, 2, 3, 4}; // not allowed
```

初始化数组时，提供的值少于数组的长度，则数组其余部分会被设置为0

```
// 将数组全部元素设置为0
int a[10] = {0};
```

如果`arraySize`为空，数组长度为初始化的元素列表长度

```cpp
// void init_2d_array(float mat[][], size_t rows, size_t cols) // error
void init_2d_array(float mat[][3], size_t rows, size_t cols);
```



### 字符串

使用`char []`存储的字符串，字符串以`\0`结尾。要注意数组长度应包含结尾的`\0`

``` cpp
char str[6] = "hello";

char str[6] = { 'h', 'e', 'l', 'l', 'o', '\0' };

char str[] = "hello"; // sizeof(str) = 6

char str[6];
str = "hello" // Not allowed
```

C++允许用空白(space, tab, 换行符)连接字符串

```cpp
char str[] = "hello" " world";
//等同于
char str[] = "hello world";
```

`cin.getline(name, size)`获取一行的字符串，保存size-1长度的字符串，并丢弃`\n`

`cin.get(name, size)`行为与`cin.getline`类似，但不丢弃`\n`

`cin.get()`获取下一个字符

```cpp
cin.getline(str, 5)
// 等同于
cin.get(str, 5).get()
```

还可以用string类存储字符串，+可以拼接两个string对象，可以用=将一个字符串对象赋值给另外一个字符串对象(一个数组不能赋值给另外一个数组)

将一行读取到string对象要用`getline(cin, str)` (Defined in header `<string>`)

c++11新增了[raw string literal](https://en.cppreference.com/w/cpp/language/string_literal), 默认定界符是`"(`和`)"`在需要的情况下，还可以在引号和括号前增添任意数量字符,作为新的定界符

```cpp
cout << R"(Line feed is \n)" << endl;
cout << R"!(Default left delimiter is "()!" << endl;

// output
// Line feed is \n
// Default left delimiter is "(
```

### Structure

可用`=`把一个结构赋值给另一个同类型的结构，即使成员中有数组

```cpp
#include <iostream>

using namespace std;

struct {
	int a[10];
} x, y;

int main() {
	x = {
		{1, 2, 4, 3}
	};
	y = x;
	cout << x.a << endl;
	cout << y.a << endl;
	return 0;
}

// output
// 0x55df4bfbc180
// 0x55df4bfbc1c0
```

structure会4字节对齐

```cpp
struct Student {
	int id;
	bool male;
	char label;
	float height;
};

struct Student2 {
	int id;
	bool male;
	float height;
	char label;
};

cout << sizeof(Student) << endl; //12
cout << sizeof(Student2) << endl; //16
```

![image-20211025084639907](/home/arv/.config/Typora/typora-user-images/image-20211025084639907.png)

在c中，不能直接直接使用结构类型定义变量，而是要在前面加上struct或者使用typedef

```cpp
typedef struct _rgb_struct{
    unsigned char r;
    unsigned char g;
    unsigned char b;
} rgb_struct;

struct _rgb_struct rgb1;
rgb_struct rgb2;
```



### Union

`Union`能存储不同的数据类型，但只能同时存储其中一种类型

```cpp
#include <iostream>
#include <cstring>

using namespace std;

union id {
	int id_num;
	char id_char[20];
} id_val;

int main() {
	id_val.id_num = 111;
	cout << id_val.id_num << endl;
	strcpy(id_val.id_char, "Ji");
	cout << id_val.id_char << endl;
	cout << id_val.id_num << endl;
	return 0;
}

/* output
111
Ji
26954
*/
```

```cpp

#include <iostream>
using namespace std;

union ipv4address{
    std::uint32_t address32;
    std::uint8_t address8[4];
};

int main()
{
    union ipv4address ip;

    cout << "sizeof(ip) = " << sizeof(ip) << endl;

    ip.address8[3] = 127;
    ip.address8[2] = 0;
    ip.address8[1] = 0;
    ip.address8[0] = 1;

    cout << "The address is " ;
    cout << +ip.address8[3] << ".";
    cout << +ip.address8[2] << ".";
    cout << +ip.address8[1] << ".";
    cout << +ip.address8[0] << endl;

    cout << std::hex;
    cout << "in hex " << ip.address32 << endl;

    return 0;
}
```



### Enum

```cpp
#include <iostream>
#include <cstring>

using namespace std;

enum spectrum {
	red, orange, yellow, green
};
spectrum band;

int main() {
	cout << red << endl; 
	cout << green << endl;
	band = orange;
	cout << band << endl;
	return 0;
}

/* output
0
3
1
*/
```

enum可以提升为int类型，但int类型不能自动转换为enum，需要强制转换

可以用`enum`批量定义常量

```cpp
#include <iostream>
#include <cstring>

using namespace std;

enum spectrum {
	red, orange, yellow, green
};
enum days { monday, tuesday };

int main() {
	cout << red << endl; 
	cout << monday << endl;
	return 0;
}
/* output
0
0
*/
```

### Pointer

`*`两边的空格是可选的

```cpp
int * ptr; // valid
int* ptr; // valid
int *ptr; // valid
```

下面的声明创建了一个指针和一个int变量

```cpp
int *a, b;
```

不能简单的将整数复制给指针

```cpp
int *ptr;
ptr = 0xB8000000; // invalid
ptr = (int *) 0xB8000000; // valid
```

用`new`为指针分配内存，用`delete`释放内存

用`delete`释放内存时，指针指向的内存会被释放，但指针本身不会被删除，且指针仍然指向原内存地址。

**只能用`delete`释放`new`创建的内存**

```cpp
#include <iostream>
#include <cstring>

using namespace std;

int main() {
	int *ptr = new int;
	cout << ptr << endl;
	delete ptr;
	cout << ptr << endl;
}

/* output
0x55a1e79aceb0
0x55a1e79aceb0
*/
```

使用`new`创建动态数组，`new`远算符返回第一个元素的地址

```cpp
int *psome = new int[10];
delete [] psome;
```

数组名是指向数组第一个元素的地址，且数组名是常量

```cpp
pointerName = pointerName + 1; // Valid
arrayName = arrayName + 1; // Invalid
```

`sizeof(arrrayName)`得到的是整个数组的长度, `sizeof(pointerName)`得到的是指针的长度。

```cpp
int arr[4];
int * ptr = arr;
cout << sizeof(arr) << endl; // 16
cout << sizeof((int *) arr) << endl; // 8
cout << sizeof(ptr) << endl; // 8
```

`arrayName`是数组第一个元素的地址，而`&arrayName`是整个元素的地址。

下面的例子里，`&arrayName`对应的类型是`int (*) [4]`，即指向长度为4的int数组的指针。

```cpp
int arr[4];
cout << arr << endl;
cout << arr + 1 << endl;
cout << &arr + 1 << endl;
int (*ptr)[4] = &arr;
cout << ptr + 1 << endl;

/** output
0x7ffccef0af00
0x7ffccef0af04
0x7ffccef0af10
0x7ffccef0af10
*/
```

解除引用有两种方法，一种是用`*`, 另一种使用数组表示法, 如`ptr[0]`

> **在cout和多数c++表达式中, char数组名, char指针以及用引号括起的字符串常量都被解释为字符串第一个字符的地址**

```cpp
*pd[3] // an array of 3 pointers
(*pd)[3] // a pointer to an array of 3 elements
```



箭头成员运算符`->`用于指针

```cpp
struct things {
    int good;
    int bad;
};
things grubnose = { 1, 2 };
things * ptr = &grubnose;
cout << (*ptr).good << endl;
cout << ptr->bad << endl;
```

c++有三种管理数据内存的方式：自动存储(函数内部定义的变量), 静态存储(函数外部定义的变量或static), 动态存储(new创建)

### 引用变量 Reference

引用在C++中才有，C里没有。

引用是一个已经定义的变量的别名。

```cpp
int a = 33;
int & b = a;
cout << &a << endl;
cout << &b << endl;

// 0x7ffff1d13b0c
// 0x7ffff1d13b0c
```

在声明引用变量时必须初始化。在之后给引用变量赋值相当于给原变量赋值。

引用变量只能引用左值。

左值大致是指可出现在赋值语句左边的实体(const变量是特例，属于不可修改的左值)。

函数还可以返回引用。

```cpp
int & func1(int & x) {
	return x;
}

int main() {
	int a = 10;
	func1(a) = 20;
	cout << a << endl; // 20
}
```

如果不允许给返回的引用赋值，将返回类型声明为const

```cpp
const int & func1(int &x);
```



不要返回函数内部的临时变量的引用

```cpp
int & func() {
	int x = 100;
	return x; // Bad behaviour
}
```

如果引用参数是const, 则会在下面2种情况下生成临时变量

- 实参类型正确，但不是左值
- 实参类型不正确，但可以转换为正确的类型

```cpp
void func(int &a) {} // Error
void func(const int &a) {} // Ok
func(2);
```



### 右值引用 rvalue reference (c++11)

右值包括字面值，表达式，和常规函数的返回值。

```cpp
int && a = 100;
int b = 20;
int && c = 2*b + 3;
```

## Corner cases

### Dangling else

> In C++, the else is associated with the closest if that doesn't have an else.

```cpp
int num = 11;
if(num < 10)
	if(num < 5)
		cout << "The number is less than 5" << endl;
else
	cout << "Where I'm?" << endl;
// output nothing!
```

### Order of evaluation

https://en.cppreference.com/w/cpp/language/eval_order

> There is no concept of left-to-right or right-to-left evaluation in C++. This is not to be confused with left-to-right and right-to-left associativity of operators: the expression `a() + b() + c()` is parsed as `(a() + b()) + c()` due to left-to-right associativity of operator+, but **the function call to `c` may be evaluated first, last, or between `a()` or `b()` at run time**

v = value computation s = side effect (since c++11)

- For any operator, v(operands) > v(result) (2)
- For post-increment and post-decrement operators, v > s
- For pre-increment and pre-decrement operators, s > v
- For `&&` and `||` operator, s,v(left argument) > s,v(right argument).
- For all built-in assignment operators and all built-in compound assignment operators, v(left argument) > s(assignment) ,v(right argument) > s(assignment) and s(assignment) > v(assignment)
- 



## STL

### vector

### array

array比vector效率更高，array的长度是固定的。

> array对象和数组存储在相同的内存区域(栈)中，而vector对象存储在另一个区域(堆)

上面这句话，实际测试中并不适用。可能与平台有关

```cpp
vector<int> a1;
array<int, 4> a2;
int a3[4];
cout << &a1 << endl;
cout << &a2 << endl;
cout << &a3 << endl;
/* output
0x7ffe8feea5a0
0x7ffe8feea5c0
0x7ffe8feea5d0
*/
```



## 常量

可以通过`const`声明常量，常量必须在声明后立刻赋值。

### const 指针

```cpp
int a = 20;
const int * ptr = &a; 
// *ptr = 40; // Invalid
int b = 30;
ptr = &b;
cout << *ptr << endl; // 30
```

在上面的例子中, `const int * ptr`表明`ptr`指向一个`const int`，因此不能修改`*ptr`。但可以修改指针指向的地址，即`ptr`。

```cpp
int a, b;
int * const ptr = &a;
ptr = &b; // Invalid
*ptr = 30; // Ok

const int * const ptr1 = &a;
ptr1 = &b; // Invalid
*ptr1 = 30; // Invalid
```

改变const的位置，此时`ptr`是常量，`ptr`指向的地址不能被修改。

**不能将const的地址赋值给非const指针**

```cpp
const int a = 30;
int * ptr = &a; // Invalid
```

在下面的例子中，由于arr是常量, func的形参也必须加上const

```cpp
/*
void func(int arr[]) {
}
*/

void func(const int arr[]) {
}

int main() {
	const int arr[] = {1, 3, 5, 7};
	func(arr);
}
```

总而言之，当数据类型不是指针时，const地址和非const地址可以赋值给const指针。而非const指针只能指向非const地址。

数据类型为指针时，必须对应const.

## 函数

### 形参和实参

A variable that’s used to receive passed values is called a formal parameter or formal argument(or parameter for short). 

The value passed to the function is called the actual parameter or actual argument(or argument for short).

### 原型

函数原型中，必须标明参数类型，参数名是可选的。参数名相当于占位符，不必和函数定义中的参数名称相同。

```cpp
void func(int); // valid
void func(int x); // valid
```

在原型或者函数头中,`int []`与`int *`是等效的。

```cpp
void func(int arr1[], int * arr2) {
	cout << (arr1 + 1) << endl;
	cout << (arr2 + 1) << endl;
}

int main() {
	int arr[4];
	func(arr, arr);
}
/** output
0x7ffc536fb6b4
0x7ffc536fb6b4
*/
```

### 函数与二维数组

必须标明子数组的大小。

```cpp
void func2(int (*arr)[4], int rows);
void func1(int arr[][4], int rows);
```

### 函数与结构

结构是按值传递的。

### 函数指针

```cpp
// Type of say: void (*)(const char*)
void say(const char * str) {
	cout << str << endl;
}

void call(void (*func)(const char * str)) {
	func("Hello!");
}
```

用`(*func)`替换函数名，就是函数指针的类型。

在调用函数时，既可以使用`func()`,也可以使用`(*func)()`

在给函数指针赋值时，即可以使用取地址运算符`&`也可以不使用。

```cpp
// Improve your understanding of function pointer!

const double * f1(const double ar[], int n);
const double * f2(const double [], int);
const double * f3(const double *, int);

int main() {
	const double * (*p1)(const double *, int) = f1;
	auto p2 = f2;
	
	const double * (*pa[3])(const double *, int) = {f1, f2, f3};
	const double * (**pb)(const double *, int) = pa;
	const double * (*(*pc)[3])(const double *, int) = &pa;
}
```

使用`typedef`简化类型

```cpp
typedef const double * (*fptr)(const double *, int);
fptr p1 = f1; 
```

```cpp
float norm_l1(float x, float y);
float norm_l2(float x, float y); 
float (*norm_ptr)(float x, float y);

norm_ptr = norm_l1; //Pointer norm_ptr is pointing to norm_l1

norm_ptr = &norm_l2; //Pointer norm_ptr is pointing to norm_l1

float len1 = norm_ptr(-3.0f, 4.0f); //function invoked 

float len2 = (*norm_ptr)(-3.0f, 4.0f); //function invoked
```



### 函数引用

和普通的引用相同，初始化时必须赋值。

```cpp
void (&norm_ref)(int) = func<int>;
```



### 内联函数

内联函数用内敛代码替换函数调用。

内联函数不能递归。

```cpp
inline double square(int x) {
    return x*x;
}
```

或者使用宏达到相同的效果

```cpp
#define SQUARE(X) ((X)*(X))
```

1e9次调用`square`，结果为宏函数(1.667s) > 内联函数(2.017s) > 普通函数(2.047s)

### 默认参数

A feature in C++ (not C).

> In a function declaration, after a parameter with a default argument, all subsequent parameters must **have a default argument supplied in this or a previous declaration** from the same scope.

如果要为某个参数设置默认值，则该参数右面的所有参数也必须有默认值。

```cpp
void func(int a, int b, int c = 1, int d); // Invalid
void func(int a, int b, int c = 1, int d = 1); // Valid
```

如果函数声明和原型都存在，默认参数只能放在两者之一。

在函数原型中，可以分别设置默认参数，但要保证该参数后面的参数已经都有默认值。

```cpp
float norm(float x, float y, float z);
float norm(float x, float y, float z = 0);
float norm(float x, float y = 0, float z);

float norm(float x, float y, float z)
{
    return sqrt(x * x + y * y + z * z);
}
```



### 函数重载 Overload

匹配函数时，参数不区单纯的引用。

```cpp
// They are the same.

void func(const int a);
void func(int a);
```

对于非指针非引用类型，不区分加不加const

```cpp
void func(char * str){
	cout << "char" << endl;
}

void func(const char * str) {
	cout << "const char" << endl;
}


void func(const int & a) {
	cout << "const int" << endl;
}

void func(int & a) {
	cout << "int" << endl;
}

const char * p1 = "sss";
char p2[] = "aaa";
func(p1); // const char
func(p2); // char
int a = 123;
const int b = 124;
func(a); // int
func(b); // const int

```

在上面的例子中，`void func(const char * str)`必须存在，因为不能将const指针赋值给非const指针。

如果有多个匹配的函数版本，将选择最匹配的版本。

### 函数模板

#### Explicit specialization

显式具体化(explicit specialization)，为某个特定类型编写函数。

在使用显式具体化之前，必须有可供"覆盖"的模板函数。

优先级：普通函数 > explicit specialization > 模板函数

```cpp
template <typename T>
void say(T x) {
	cout << x << endl;
}

template <> void say<Person> (Person x) {
	cout << x.name << endl;
}
```

#### Explicit instantiation

定义模板后，编译器会按需生成的函数定义。如果要强制生成某种类型的函数定义，可以使用显式实例化。

显式实例化关键词是`template`，注意没有`<>`

```cpp
// explicit instantiation

template void swap<int>(int, int); 
template void swap<>(char, char);
```

```cpp
template <typename T>
T Add(T a, T b) { return a + b; }

int m = 6;
double x = 10.2;
Add<double>(x, m); // explicit instantiation
```

#### 强制使用模板

如果模板函数和非模板函数都匹配，而我们想调用非模板函数，可以这么做

```cpp
template <typename T>
void say(T x) {
	cout << "SAY" << endl;
}

void say(char * str) {
	cout << str << endl;
}

char str[] = "hello";
say<>(str); // SAY
```

### decltype(c++11)

`decltype`是c++11新增的特性，用于确定类型

`decltype`推断类型的步骤如下

> 1. If `expression` is an unparenthesized identifier (that is, no additional parentheses), then `var` is of the same type as the identifier, including qualifiers such as `const`.
> 2. If `expression` is a function call, then var has the type of the function return type. (The function won't be called.)
> 3. If `expression` is an lvalue, then `var` is a reference to the expression type.
> 4. If none of the preceding special cases apply, `var` is of the same type as `expression`

```cpp
double a, b;
cout << &b << endl;
decltype(a) x = b;
cout << &x << endl;
decltype((a)) y = b;
cout << &y << endl;

/*
0x7ffc215348c0
0x7ffc215348c8
0x7ffc215348c0
*/
```

### 后置返回类型(c++11)

```cpp
template<typename T1, typename T2>
decltype(x+y) func(T1 x, T2 y); // Invalid
```

上面这段代码是非法的，因为在使用`decltype`的时候还没有声明`x`和`y`。这种情况下，可以使用后置返回类型。

```cpp
template<typename T1, typename T2>
auto func(T1 x, T2 y) -> decltype(x+y);
```

`auto`相当于占位符，表示实际的返回类型在后面。

## IO

`cin.get(ch)`和`ch = cin.get()`会读取所有的输入字符(包括空格、回车)。

`cin >> ch`不会读取空白字符

`os.setf()`和`os.precision()`的设置是持久的，`os.width()`只会对下一次输出生效。

`scanf` uses whitespace—spaces, tabs, and newlines to separate a string. `gets()` stops reading input when it encounters a newline or End of file.

`getline()`and `get()` both read an entire input line—that is, up until a newline character. However, `getline()` discard the newline character, whereas `get()` leave it in the input queue.







## 类型转换

### Implicit conversion

https://en.cppreference.com/w/cpp/language/implicit_conversion

```cpp
unsigned char a = 255;
unsigned char b = 1;
int c = a+b; // 256 
```

The operands will be converted to one of the four types without losing data: int, long, float, double.

```
float f = 17/5; // 3.f
float f = 17.f/5; // 3.4f
float f = 17/5.f // 3.4f
```



### 强制类型转换

```
(typename) value // c style

typename(value) // c++ style
```

## Expression

表达式都有值。赋值表达式的值为成员左侧的值

```cpp
int a;
int b = (a = 9) + 8;
cout << b << endl; //17
```

### typeid

```cpp
template<typename T>
void func(T x) {
	cout << typeid(T).name() << endl;
}
```

### typedef

```cpp
typedef unsigned char vec3b[3];

unsigned char color[3];
vec3b color = {255, 0, 255}; 
```

### sizeof

`sizeof` on array function parameter will return byte size of pointer of the corresponding type.

```cpp
void sum_arr(int arr[]) {
	cout << sizeof arr << endl; // =sizeof(int *)
}
```



## 类和对象

类成员的默认访问类型是`private`；结构体的默认访问类型是`public`

在类中定义的方法都将自动成为内联函数(加上`inline`)

一个类的实例共享该类的方法。在创建一个实例时，不会创建方法的副本。

### Constructor

使用构造函数

```cpp
Person p = Person(30, "Jack", true);
Person p(30, "Jack", true);
Person p = {30, "Jack", true};
Person p{30, "Jack", true};

Person * p = new Person(30, "Jack", true);
Person * p = new Person{30, "Jack", true};
```

只有在**没有定义任何构造函数**时，编译器才会提供默认构造函数。默认构造函数不处理任何事情。相当于

```cpp
class Person {}
//default constructor is as Person::Person{} 
Person p; // calls default constructor implicitly
```

调用默认函数函数时不要加`()`，加上括号时声明了一个函数

```cpp
Person p(); //declare a function that returns an instance of Person and has no argument
```

![image-20220109152704671](/home/arv/.config/Typora/typora-user-images/image-20220109152704671.png)

`const` member variables can be initialized by initialization list.

#### 初始化列表

列表初始化,只有构造函数能使用这种语法

```cpp
Classy::Classy(int n, int m) :mem1(n), mem2(0), mem3(n*m+2){}
```

const、引用数据成员必须通过列表初始化或类内初始化完成初始化。

初始化的顺序为它们被声明的顺序，而不是它们在初始化列表中的顺序。

类内初始化

```cpp
class Queue {
    Node * front = NULL;
    Node * rear = NULL;
    int items = 0;
    ...
}
```



### Destructor

析构函数的名称是`~`加上类名

### Member access

#### Public

#### Protected

> A protected member of a class is only accessible
>
> 1. to the members and friends of that class;
> 2.  to the members *and friends (until C++17)* of any **derived class** of that class, but only when the class of **the object through which the protected member is accessed is that derived class or a derived class of that derived class**

```cpp
class Base 
{
  protected:
    int n;
  private:
    void foo1(Base& b)
    {
        n++; // Okay
        b.n++; // Okay
    }
};

class Derived : public Base 
{
    void foo2(Base& b, Derived& d) 
    {
        n++; //Okay
        //b.n++; //Error. 
        d.n++; //Okay
    }
};

```

#### Private

Private members are only accessible within the class defining them.

A private member of a class is only accessible to the members and friends of that class, **regardless of whether the members are on the same or different instances**.

### const member function

下面这段代码会出错.因为p是个常量，编译器不知道调用`sayName`是否会修改p内部的变量。

```cpp
void Person::sayName(){
	cout << "Jack" << endl;
}

const Person p;
p.sayName(); //Error
```

这时候就需要const成员函数了。就是将`const`放在函数声明定义的括号后面。

```cpp
void sayName() const;

void Person::sayName() const{
	cout << "Jack" << endl;
}
```

A const member function defined outside the class body must specify the const modifier in **both its declaration and definition. **

Const member function can change the value of static member variables.

### 类枚举

下面的代码会冲突

```cpp
enum t_shirt {SMALL, MEDIUM, LARGE};
enum egg_old {SMALL, MEDIUM, LARGE};
```

使用作用域为类的枚举(c++11)，避免这种冲突

```cpp
enum class t_shirt {SMALL, MEDIUM, LARGE};
enum class egg_old {SMALL, MEDIUM, LARGE};
t_shirt size = t_shirt::SMALL;
```

### 类常量

```cpp
enum {Lbs_per_stn = 14};
static const int Lbs_per_stn = 14; //alternatively
```



### 运算符重载

```cpp
Time t1, t2, t3, t4;

// two equal statements
t3 = t1 + t2; 
t3 = t1.operator+(t2);

// what is t1 + t2 + t3
t4 = t1 + t2 + t3; // = (t1+t2)+t3
t4 = t1.operator+(t2 + t3);
t4 = t1.operator+(t2.operator+(t3));
```

重载限制

- 不能创建新的运算符

- 不能为标准类型重载运算符（至少一个操作数是自定义类型）

- 某些运算符不能被重载

- 大多数运算符都可以通过成员或非成员函数进行重载，但下面的运算符只能通过成员函数进行重载。

  ![image-20211108184944365](/home/arv/.config/Typora/typora-user-images/image-20211108184944365.png)

为了避免出现`force1 + force2 = net`的情况，重载运算符的返回类型应该加上`const`

`const Vector operator+(const Vector & f1, const Vector & f2) const;`

```cpp
// prefix increment
MyTime& operator++()
{
    this->minutes++;
    this->hours += this->minutes / 60;
    this->minutes = this->minutes % 60;
    return *this; 
}
// postfix increment
MyTime operator++(int)
{
    MyTime old = *this; // keep the old value
    operator++(); // prefix increment
    return old; 
}
```



### 友元函数

通过让函数成为类的友元，可以赋予该函数与类的成员函数相同的访问权限。

```cpp
A = B * 2.75;
// is converted to
A = B.operator*(2.75);

// But what if
A = 2.75*B;
```

这时我们就需要非成员重载运算符。但非成员函数不能访问类的私有数据。于是，友元函数派上用场了。

创建友元函数：将函数原型放在类中，并在原型前加`friend`

演示

```cpp
class Complex {
	double realPart, imaginaryPart;
	public:
		Complex(double r = 0, double i = 0):realPart(r), imaginaryPart(i) {}
		Complex operator*(double x) const;
        void display();
        friend Complex operator*(double, const Complex &);
};

Complex operator*(double x, const Complex & y) {
	return Complex(y.realPart * x, y.imaginaryPart * x);
}

// Alternatively, without friend function.
Complex operator*(double x, const Complex & y) {
    return y * x;
}

Complex c5 = 3*c1; 
```

*`=`(赋值运算符)、`()`、`[]`、`->`只能通过成员函数重载，不能用友元函数重载。

重载<<运算符用于输出, `ostream`不用加`const`

```cpp
friend ostream & operator<<(ostream & os, const c_name & obj);
```



### 类的类型转换

- 构造函数
- 转换函数

可以定义只接受一个参数(或多个参数，其他参数有默认值)的构造函数。这样，类可以隐式或显式地为转换参数的类型。

当使用`explicit`限定构造函数的原型时，只允许显式转换。

```cpp
class Time {
	int hours, minutes, seconds;
	public:
		Time(int x) {
			hours = x / 3600;
			x %= 3600;
			minutes = x / 60;
			seconds = x % 60;
		}
		Time(int h, int m, int s):hours(h), minutes(m), seconds(s) {}
		Time(){}
		
		void display() const {
			printf("%dh %dmin %ds\n", hours, minutes, seconds);
		}
};

Time t1 = 3600; // Time t1 = Time(3600)
Time t2;
t2 = 4900; // t2 = Time(4900)
```

加上`explicit`后

```cpp
class Time {
	...
	explicit Time(int x) {...}
	...
}

Time t1 = (Time)3600; // explicit conversion
```

构造函数可以将其他类型转换为对象。如果要将对象转换为其他类型，需要转换函数。

```
operator typeName();
```

- 转换函数没有返回类型
- 转换函数没有参数

同样的，在转换函数的原型前加`explicit`，也能禁止隐式转换。(c++11)

### Static member variable

> 不能在类声明中初始化静态成员变量，这是因为声明描述了如何分配内存，但并不分配内存。
>
> 需要在类声明之外使用单独的语句来进行初始化。静态类成员是单独存储的，而不是对象的组成部分。

否则会出现link error

### Static member function

普通函数前面加`static`，成为静态成员函数。

静态成员函数不和类的实例关联，只能访问静态成员。

### 内存管理

c++自动生成了下列成员函数（如果没有手动定义）

- 默认构造函数
- 默认析构函数
- 复制构造函数 copy constructor， 复制非静态成员
  `Class_name(const Class_name &)`
- 赋值运算符 assignment operator
  `Class_name & Class_name::operator=(const Class_name &)`
- 地址运算符

在类中使用动态内存时，要着重考虑copy constructor和assignment operator。

如果使用基类和子类都用到了动态内存分配，那么除了基类和子类要定义相应的复制构造函数，赋值运算符和析构函数外，还要记得**在子类的这些函数中调用父类相应的函数**。其中，copy constructor和assignment operator要显式调用。

Tips

- 复制构造函数 深拷贝？浅拷贝？注意和destructor的联系
- 赋值运算符 可能要**释放之前分配的内存**
- 赋值运算符 返回引用
- 赋值运算符要避免将自身赋给自身
  `if (this == &obj) return *this;`
- 将指针初始化为NULL或nullptr
- new分配内存要记得释放

![image-20211231092122041](/home/arv/.config/Typora/typora-user-images/image-20211231092122041.png)

### 继承

派生类不能直接访问基类的私有成员，而必须通过基类方法进行访问。

派生类构造函数必须使用基类构造函数(使用列表初始化)，如果没有显式调用，将自动调用基类的默认构造函数.

基类指针可以在不进行显式类型转换的情况下指向派生类对象；基类引用可以在不进行显式类型转换的情况下引用派生类对象。然而，基类指针或引用只能用于调用基类方法

For structures, the default behavior is public inheritance, whereas for class is private inheritance.

![image-20220106143537612](/home/arv/.config/Typora/typora-user-images/image-20220106143537612.png)

如果希望基类的方法在派生类外可用，可以考虑的一种方法是使用`using`

```cpp
class A{
	public:
	void say() {
		cout << "A" << endl;
	}
};

class B : private A{
	public:
	using A::say;
};

int main() {
	B b;
	b.say(); // A
}
```



### virtual function

如果没有用`virtual`，程序根据引用或指针类型判断使用哪个方法。

如果使用`virtual`，程序根据引用或指针**指向的对象的类型**选择使用哪个方法。

方法在基类中被声明为虚的后，它在派生类中将自动成为虚方法。

```cpp
class A{
	public:
	virtual void say() {
		cout << "A" << endl;
	}
};

class B : public A{
	public:
	void say() {
		cout << "B" << endl;
	}
};

class C : public B{
	public:
	void say() {
		cout << "C" << endl;
	}
};

int main() {
	C c_obj;
	A& c_ref = c_obj;
	c_ref.say(); // C
}
```

**建议添加虚析构函数。**否则，在使用多态时，虚构函数可能不会按照正确的顺序执行。(析构函数中有内存释放等)

#### vtbl

编译器通过virtual function table, vtbl来实现虚函数。

每一个对象都有一个隐藏的数组，数组保存了虚函数的地址。如果派生类没有提供虚函数的定义，数组中保存原始函数的地址。如果提供，该函数的地址也被添加到数组中。

#### pure virtual function

```cpp
class BaseEllipse {
    public:
    virtual ~BaseEllipse(){}
    virtual double Area() const = 0; // a pure virtual function
}
```

如果类中包含纯虚函数，则不能创建该类的对象。

### virtual base class

> 虚基类使得从多个类（它们的基类相同）派生出的对象只继承一个基类对象。

例如，

```cpp
class Singer : virtual public Worker {...}
class Waiter : public virtual Worker {...}
class SingingWaiter : public Singer, public Waiter {...}
```

此时`SingerWaiter`对象只包含一个`Worker`对象。

### 类模板

不能简单地将模板类成员函数的定义放在独立的文件中。因为函数模板并不是真正的函数，不能单独编译。

可以在类中简写类型，如`Stack& operator=(const Stack & st)`

但在类外面必须写全，`Stack`必须换成`Stack<Type>`

#### Non-type parameters

```cpp
template<typename T, size_t n>
class ArrayTP {
    private:
    T arr[n];
    
    public:
    ArrayTP(){}
    explicit ArrayTP(const T & v);
    virtual T & operator[](size_t i);
    virtual T operator[](size_t i) const;
    
};
```

#### 默认值

可以为模板参数(类型参数、非类型参数)提供默认值

```cpp
template <typename T1, typename T2 = int>
class Topo {...}
```

#### 参数作为模板

```cpp
template <template <typename T> class Thing>
class Crab{
    Thing<int> s1;
    ...
};

Crab<Stack> container;
```

在上面的例子中,`container`中的`Thing<int>`会被替换为`Stack<int>`.

`Stack`必须也是一个模板类，并且和`template <typename T> class Stack{...}`相匹配

更一般的例子

```cpp
template <template <typename T> class Thing, typename U, typename V>
class Crab {
    Thing<U> s1;
    Thing<V> s2;
}
```

#### 友元函数

```cpp
template <typename T>
class HasFriend {
    friend void report1();
    friend void report2(HasFriend<T> &);
}
```

如果友元函数自身也是模板函数

```cpp
template <typename T> void counts();
template <typename T> void report(T &);

template <typename T> class HasFriend{
    friend void counts<T>();
    friend void report<>(HasFriend<T> &);
    // Alternatively
    // friend void report<HasFriend<T>>(HasFriend<T> &) 
}
```

非约束友元。每个函数模板具体化都是该类具体化的友元。

```cpp
template <typename T>
class ManyFriend {
    template <typename C, typename D>
    friend void show2(C &, D &);
}

template <typename C, typename D>
void show2(C & c, D & d) {
    ...
}
```

### 友元类

友元类可以访问原始类的保护、私有部分。

```cpp
class TV {
    friend class Remote;
    ...
};

class Remote {
    ...
}
```

友元声明放在`public` ,`protected`或`private`部分都是可以的，位置无关紧要。

### Forward declaration

在下面的例子里，

```cpp
class TV {
    friend void Remote::setChannel(TV & t, int c);
}
```

`TV`声明了一个友元函数，编译器需要知道`Remote`里是否有`setChannel`这个函数，因此`Remote`定义要放在`TV`前。但`Remote`的方法又用到了`TV`.这就造成了循环依赖。

解决方法是使用向前声明(Forward declaration)

```cpp
class TV; // forward declaration
class Remote {...}
class TV {...}
```

如果`Remote`中使用了`TV`的某个成员，那么需要保留函数原型，将函数定义写在`TV`类后面。

另外,`friend class Remote`除了指出`Remote`是一个友元类外，还表名它是一个类，有向前声明的功能。

## RTTI

Runtime Type Identification(RTTI).

The intent of RTTI is to provide a standard way for a program to determine the type of object during runtime. 

RTTI is provided through two operators:

- `typeid` operator, which returns the type of a given expression
- `dynamic_cast` operator, which safely converts a pointer or reference to a base type into a pointer of reference to a derived type.

RTTI只适用于包含虚函数的类。

```cpp
class Base{
	virtual void func(){};
};
class Derived: public Base{};

Base * bp = new Derived;
Derived * dp = new Derived;
if(typeid(*bp) == typeid(*dp)) {
    cout << "Ok2" << endl; 
}

// If the virtual function in Base class were commented, then 'Ok2' would not be printed out.
```



## 类型转换运算符

### dynamic_cast

The `dynamic_cast` operator is used to **safely** convert a pointer or reference to a base class to a pointer or reference to a derived class, and use a pointer or reference to a derived class to call a non-virtual function. 

If a `dynamic_cast` to a pointer type fails, the result is 0(NULL). If a `dynamic_cast` to a reference type fails, the operator throws an exception of type `bad_cast`.

```cpp
// There must be at least one virtual function in the class B, otherwise it fails to compile.
class B { ... };
class D : public B { ... };
void f(){
   B* pb = new D;                     
   B* pb2 = new B;
   D* pd = dynamic_cast<D*>(pb);      // ok: pb points to D
   ...
   D* pd2 = dynamic_cast<D*>(pb2);   // fail,pb2 points to B not D
                                     //  pd2 is NULL
   ...
}
```



### const_cast

`const_cast` may be used to add or remove constness or volatility.

It is undefined behavior to modify a value which is initially declared as `const`.

### static_cast

It’s valid only if `type_name` can be converted implicitly to the same type that expression has, or vice versa.

### reinterpret_cast

## Smart pointers

智能指针只能用于堆内存

## 异常

### abort()

`abort()`位于`cstdlib`中。

> 其典型实现是向标准错误流（即`cerr`使用的错误流）发送消息abnormal program termination（程序异常终止），然后终止程序。

### assert

`assert` is a function-like macro in `<assert.h>` and `<cassert>`.

If `NDEBUG` is defined, do nothing.

If the condition is true, do nothing. 

Otherwise, outputs diagnostic information and call `abort()`.

### try...catch

```cpp
try {
} catch(...) {}
```

`catch(...)`可以捕获任何异常

可以使用引用、指针捕获基类异常。另外，`throw`返回的都是拷贝。

### exception类

`exception`头文件定义了`exception`类，可以把它作为其他异常类的基类。

### nothrow

`new`申请内存失败，会抛出`bad_alloc`错误。

如果希望失败时不抛出异常，而是返回一个空指针，那么要配合`nothrow`使用

```cpp
int * ptr = new (std::nothrow) int;
```



## 头文件

头文件通常包括的内容

- 函数原型
- `#define` 或 `const` 定义的符号常量
- 结构声明
- 类声明
- 模板声明
- 内联函数

## 编译

在`main.cpp`中调用了`function.cpp`中的`say()`

```cpp
#include "function.hpp"

int main() {
	say();
	return 0;
}
```

不需要include`function.cpp`，只需要引入头文件就可以。

然后编译`function.cpp`和`main.cpp`，最后将其链接。

`g++ -c`只编译不链接，生成目标文件`*.o`

### Shared libraries

Shared library in linux are `.so` files.

编译选项`-shared -fPIC`

```
g++ function.cpp -shared -fPIC -o libfunction.so
```

使用共享函数库，编译时需要加上两个选项`-L -l`

`-L`后面接着共享库所在目录。`-l`后面紧接着(不用加空格)共享库名称，省略前缀`lib`和后缀`.so`

```
g++ main.cpp -L . -lfunction
```

另外，还要配置`LD_LIBRARY_PATH`环境变量，以便程序执行时能找到共享函数库

```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:.
./a.out
```



## 存储

### Storage class specifier & cv-quanlifier

存储说明符, storage class specifier

- auto (before c++11)
- register
- static
- extern
- thread_local (since c++11)
- mutable

`mutable`指出即使类或结构变量为`const`，仍然可以修改它的某个成员。

```cpp
struct Person {
	char name[50];
	mutable int age;
};

const Person p = { "David", 30 };

int main() {
	p.age = 50; // ok!
}
```

cv-quanlifier

- const
- volatile

`volatile`表明即使程序没有对内存单元进行修改，变量值也可能发生变化。这样的场景可能出现在指针指向某个硬件端口的时候。`volatile`方便编译器进行优化。

### 作用域

局部作用域的变量只在代码块(`{}`包围的块)中可用。

### register

`register`用于建议编译器用寄存器存储变量。

`register`只能用于自动变量。

在`c++11`后，`register`关键词被保留，但已经没有了提示功能。

### 静态持续变量

静态持续变量如果没有被初始化，其所有位会被设置为0。(包括数组、结构)。

静态持续变量的三种链接性

- 外部链接性(可在其他文件中访问)
- 内部链接性(只能在当前文件中访问)
- 无链接性(只能在当前函数或代码块中访问)

```cpp
int x; // external linkage
static int y; // internal linkage

void func() {
    static int z; // no linkage
}

int main() {}
```

`static`用于代码块内表示无链接性，用于外部表示内部链接性。

### extern

不能直接使用外部变量，要先进行引用声明(referencing declaration)。引用声明使用关键字`extern`

```cpp
// main.cpp
#include <iostream>

using namespace std;
extern int x;

int main() {
	cout << x << endl;
}

// var.cpp
int x = 30;
```

将`main.cpp`和`var.cpp`编译链接(不需要include)，输出结果为30.

`extern`的声明不能初始化。

### static

下面的代码是非法的，因为在一个程序中定义了多个同名变量。

```cpp
// main.cpp
int x;
int main() {}

// var.cpp
int x;
```

可以用`static`设置`var.cpp`中`x`的内部链接性，这样就可以顺利编译链接。

```cpp
// var.cpp
static int x;
```

还可以用`static`设置函数的内部链接性，使之只能在一个文件中使用。原型和定义中都要有`static`

```cpp
static void func();
static void func() {}
```

### const

c++的全局变量默认是外部链接性的。

但如果全局变量是`const`，默认是内部链接性(相当于使用了static).

如果希望为外部链接性，需要使用`extern`关键词

```cpp
extern const int x = 30;
```

### 动态内存

初始化动态分配内存的变量

```cpp
int * x = new int(6);

// since c++11
struct Person {
    int age;
    string name;
};

Person * p = new Person { 20, "Dave" };
int * y = new int {7};
int * arr = new int[4] {1, 2, 4, 5};
```

### 定位new

普通new在堆中申请内存，而定位new可以让你使用指定位置的内存。

使用定位new需要引入`<new>`

```cpp
char mem[50];
double * arr = new (mem) double[4] {1.3, 2.5, 3.2, 10.0};

cout << arr << endl;
cout << (void *) mem << endl;

// 0x55fc5596f160
// 0x55fc5596f160
```

`delete`不能释放定位new创建的变量。

### malloc

Allocate size bytes of uninitialized storage.

```
void * malloc( size_t size );
void free(void * ptr);
```

```
int * p1 = (int*) malloc(4);
```

### new

分配内存并初始化

```cpp
//allocate an int, default initializer (do nothing)
int * p1 = new int; 
//allocate an int, initialized to 0
int * p2 = new int();
//allocate an int, initialized to 5
int * p3 = new int(5); 
//allocate an int, initialized to 0
int * p4 = new int{};//C++11 
//allocate an int, initialized to 5
int * p5 = new int {5};//C++11

Student * ps2 = new Student {"Yu", 2020, 1}; //C++11

//allocate 16 int, default initializer (do nothing) 
int * pa1 = new int[16];
//allocate 16 int, zero initialized 
int * pa2 = new int[16]();
//allocate 16 int, zero initialized 
int * pa3 = new int[16]{}; //C++11
//allocate 16 int, the first 3 element are initialized to 1,2,3, the rest 0
int * pa4 = new int[16]{1,2,3}; //C++11
```

`new`申请的内存用`delete`释放，`new []`申请的用`delete []`释放。

## Namespace

使用`::`运算符访问命名空间里的名称

```cpp
namespace Jack {
	void say() {
		std::cout << "hello";
	}
}

int main() {
	Jack::say();
}
```

使用`using`声明将命名空间中的某个名称加入当前作用域中

```cpp
int main() {
	{
		using Jack::say;
		say();
	}
	say(); //Error
}
```

还可以使用using编译指令将命名空间的所有名称导入当前作用域

```cpp
{
    using namespace Jack;
    say();
}
say(); //Error
```

命名空间还可以嵌套

```cpp
namespace Jack {
	namespace Autumn {
		int score;
	}
}

Jack::Autumn::score = 70;
```

命名空间中也可以使用using编译指令和using声明。

```cpp
namespace Jack {
    using namespace std;
}
```

还可以给命名空间添加别名

```cpp
namespace Bill = Jack;
```

多个文件的命名空间是共通的。

### using声明与编译指令的区别

如果命名空间和**声明区域(当前作用域)**同时定义了某个名称，那么在用using声明导入相同的名称时，会出错。

```cpp
namespace Jack {
	void say() {
		std::cout << "hello";
	}
	int count;
	double pi;
}

int count;

int main() {
	double pi;
	using Jack::count; //Ok
	using Jack::pi; //Error
}
```

在上面的例子中，`count`不是定义在main函数的作用域，所以不会出错。

using编译指令将编译指令视为在函数之外声明的。局部版本会覆盖命名空间的版本。

```cpp
int main() {
	double pi = 3.1415;
	using namespace Jack;
	std::cout << pi; //3.1415
}
```



## 预编译指令

### #ifndef

`ifndef`常用于防止某个文件被引入多次

例如，有3个源文件: `main.cpp` `function.cpp` `header.hpp` .

 `main.cpp`和`function.cpp`都include了`header.hpp`。 此外,`main.cpp`include了`function.cpp`

这样，就导致了`main.cpp`重复include`header.hpp`，编译会出错。

解决方式之一是在`header.hpp`中定义一个常量，判断是否已经被引入。

```cpp
#ifndef __DRAW_H__
#define __DRAW_H__
bool drawLine(int x1, int y1, int x2, int y2);
bool drawRectangle(int x1, int y1, int x2, int y2);
#endif
```

### #pragma once

也可以使用`#pragma once`来防止某个文件被Include多次。

其作用于整个文件，而不是某段代码。

## Make

### Rules

A makefile consists of a set of rules.

The template of rules is as follows.

```
<target>: <prerequisites>
[tab]	<commands>
```

- target is the name of file to be generated.
- prerequisites are file names, separated by spaces, as input to create the target.
- commands must start with tab.

### Target

If you don't specific target when running make, make will run the first rule in makefile.

And if any of the prerequisites is modified or does not exists, make will turn to the rule whose target is equal to the name of the prerequisites if exists, and fail otherwise. 

![image-20210925101942086](https://raw.githubusercontent.com/143kk/blog-images/master/img/image-20210925101942086.png)

Use `.PHONY` when you want to create a make command rather than a real target.

```
.PHONY: clean
clean: 
	rm -f *.o $(TARGET)
```

Adding `.PHONY` in the example above allows you to execute `make clean` in the shell successful when there exists a file named `clean` up to date in the current directory. 

Otherwise, you will receive an error like this:  "make: 'clean' is up to date."

### Comments

comments start with `#`

### Variables

You can define and use variables in make.

Also, there are some built-in variables in make, such as `CXX`, which is `g++` by default. For more details, please see https://www.gnu.org/software/make/manual/html_node/Implicit-Variables.html

```makefile
TARGET = a
OBJ = f.cpp

$(TARGET): $(OBJ)
	$(CXX) -o $(TARGET) $(OBJ)
```

### Automatic variables

- `$@` Object files
- `$^` all the prerequisites files, separated by space
- `$<` the first prerequisites file

```makefile
TARGET = hello
OBJ = main.o printhello.o factorial.o

CXXFLAGES = -c -Wall

$(TARGET): $(OBJ)
	$(CXX) -o $@ $(OBJ)
	
%.o: %.cpp
	$(CXX) $(CXXFLAGES) $< -o $@
```

### Functions

```makefile
SRC = $(wildcard ./*.cpp)

target:
	@echo $(SRC)
```



----

参考资料

Lab03.pptx (From the lesson *C/C++ Program Design*)

[Make命令教程](https://www.ruanyifeng.com/blog/2015/02/make.html)

[GNU Make Manual](http://www.gnu.org/software/make/manual/make.html)

## CMake

CMake是一个跨平台的编译工具。在Linux环境下，运行CMake会生成makefile。然后再运行make进行编译。

CMake对应的文件是`CMakeLists.txt`

```shell
cmake .
make
```

常用命令

```cmake
cmake_minium_required(VERSION 3.16)

project(hello) # Defines project name

# The first parameter indicates the filename of the executeable file
# The remaining parameter indicates the source files.
add_executeable(hello main.cpp)

# Collects the names of all the source files in the specified directory and stores the list in the <variable> provided.
aux_source_directory(<dir> <variable>)

# add the directory of include
include_directories(include)
```

实例

程序目录结构

```
test
    include
        function.hpp
    src
        function.cpp
        main.cpp
    CMakeLists.txt
```

```cmake
cmake_minimum_required(VERSION 3.16.3)

project(hello)

aux_source_directory(src DIR_SRCS)

include_directories(include)

add_executable(hello ${DIR_SRCS})
```

## gdb

使用gdb前，需要在编译时加上`-g`选项。

