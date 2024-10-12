---
title: "C 语言基础"
date: 2024-03-25T20:04:01+08:00
categories:
  - learn
tags:
  - C
summary: "C 语言基础及 GTK 编程。"
description: "C 语言基础及 GTK 编程。"
author: ["XiangLin"]
cover:
  image: ""
  caption: ""
  alt: ""
  relative: false
---

## C 语言快速入门

### int 类型

```c
#include <stdio.h>
#include <limits.h>
int main() {
  short int short_int = 0;
  int i = 0;
  long int long_int = 0;
  long long long_long = 0;

  printf("short int: %llu\n", sizeof(short int));
  printf("int: %llu\n", sizeof(int));
  printf("long int: %llu\n", sizeof(long int));
  printf("long long: %llu\n", sizeof(long long));

  printf("int max %d, min %d\n", INT_MAX, INT_MIN);

  unsigned int unsigned_int = 123;
  unsigned long unsigned_long = 111;

  size_t size_of_ui = sizeof(unsigned int);

  printf("unsigned int max %u\n", UINT_MAX);

  return 0;
}
```

C 语言定义了 `short int`、`int`、`long int` 和 `long long` 四种整型，以及对应的无符号整型 `unsigned int`。使用 `sizeof` 获取类型的长度，引入 `<limits.h>` 后使用常量 `INT_MAX` 等可以获取对于类型的最大值和最小值。在 MinGW 下的执行结果是：

```bash
short int: 2
int: 4
long int: 4
long long: 8
int max 2147483647, min -2147483648
unsigned int max 4294967295
```

### char 类型

```c
#include <stdio.h>
int main() {
  char a = 'a'; // 97
  char char_1 = '1'; // 49
  char char_0 = '0'; // 48

  char i = 0;

  char char_1_escape_oct = '\61';
  char char_1_escape_hex = '\x31';

  // 字面量 literal
  // \n newline
  // \b backspace
  // \r return
  // \t table
  char newline = '\n';

  printf("char a: %d\n", a);
  printf("char 1: %d\n", char_1);
  printf("char 'i': %d\n", i);

  printf("char 0: %c\n", char_0);
  printf("char 1: %c\n", char_1_escape_oct);
  printf("char 1: %c\n", char_1_escape_hex);

  // C95 宽字符
  wchar_t zhong = L'中';
  printf("中: %d\n", zhong);

  return 0;
}
```

C 语言使用 `char` 定义 ASCII 字符，使用 `wchar_t` 定义宽字符。

### 浮点型

```c
#include <stdio.h>
int main() {
  float a_float = 3.14f; // 6 10^-37
  printf("size of float: %llu\n", sizeof(float));
  double a_double = 3.14;
  printf("size of float: %llu\n", sizeof(double));

  float lat = 39.90815f;
  printf("%f\n", 39.908156f - lat);

  return 0;
}
```

### 变量

```c
#include <stdio.h>
int main(){
  int value;

  int value_init = 3;

  value = 4;
  value_init = 5;

  printf("value: %d\n", value);
  value_init = value;

  // 获取变量的地址和长度
  printf("size of value: %llu\n", sizeof(value));
  printf ("address of value: %#x\n", &value);

  return 0;
}
```

### 常量

```c
#include <stdio.h>

// 定义宏
#define COLOR_RED 0xFF0000
#define COLOR_GREEN 0x00FF00
#define COLOR_BLUE 0x0000FF

int main() {
  // 只读变量 readonly variable
  const int kRed = 0xFF0000;
  const int kGreen = 0x00FF00;
  const int kBlue = 0x0000FF;
  printf("kRed: %#x\n", kRed);

  int *p_k_red = &kRed;
  *p_k_red = 0;

  printf("kRed: %d\n", kRed);

  // macro
  printf("COLOR_RED: %d\n", COLOR_RED);
  // 取消宏定义
#undef COLOR_RED

  // 字面量 literal


  return 0;
}
```

### 运算符

```c
#include <stdio.h>
int main() {
  int first = 0;
  int second;
  int third;

  // 赋值运算符
  third = second = first;

  int left, right;
  left = 2;
  right = 3;

  // 四则运算符
  int sum = left + right;
  int remainder = left % right;

  // 关系运算符
  // true 1 false 0
  _Bool f = 3 > 1;
  printf("3 > 2: %d\n", 3 > 2);
  printf("3 < 2: %d\n", 3 < 2);

  // 逻辑运算符
  printf("3 > 2 && 3 < 2: %d\n", 3 > 2 && 3 < 2);
  printf("3 > 2 || 3 < 2: %d\n", 3 > 2 || 3 < 2);

  // 自增、自减
  int i = 1;
  int j = i++; // j = 1, i = 2;
  int k = ++i; // k = 3, i = 3;
  printf("i = %d, j = %d, k = %d\n", i, j, k);

  // 位运算符 & | ^ ~
#define FLAG_VISIBLE 0x1
#define FLAG_TRANSPARENT  0x2
#define FLAG_RESIZABLE 0x4
  int window_flags = FLAG_RESIZABLE | FLAG_TRANSPARENT;
  int resizable = window_flags & FLAG_RESIZABLE;

  return 0;
}
```

### 条件分支语句

```c

#include <stdio.h>
#include <stdbool.h>
int main() {
  _Bool is_enable = true;
  bool is_visible = false;

//  if (<condition>){
//    true statement
//  } else {
//    false statement
//  }
//
//  if (<condition>){
//    true statement
//  } else if (<condition>) {
//
//  } else {
//    false statement
//  }

#define MAGIC_NUMBER 10
  int user_input;
  printf("Please input a number: \n");
  scanf("%d", &user_input);
  if (user_input > MAGIC_NUMBER) {
    printf("You number is bigger!");
  } else if (user_input < MAGIC_NUMBER) {
    printf("You number is smaller!");
  } else {
    printf("Yes! You got it!");
  }

  // 三元表达式
  bool is_open = is_enable && is_visible ? true : false;

//  switch (<condition>) {
//    case 0 : {
//
//    }
//    break;
//    case 1: {
//
//    }
//    break;
//    default: {
//
//    }
//  }

  return 0;
}
```

引入 `<stdbool.h>` 后可以使用 `bool`、`true` 和 `false`。

### 循环语句

```c
#include <stdio.h>
#include <stdbool.h>
int main() {
  int incr = 0;
  // do-while
  do {
    if (incr++ % 2 == 0) {
      break;
    }
  } while (true);

  // while
  while (true) {
    if (incr++ % 2 == 0) {
      break;
    }
  }

  // for
  for (int i = 0; i < 10; ++i) {
    if (incr++ % 2 == 0) {
      break;
    }
  }

  return 0;
}
```

### 猜数字游戏

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <time.h>
int main() {
  srand(time(NULL));
  int magic_number = rand();
  while (true) {
    puts("Please input a number:");
    int user_input;
    scanf("%d", &user_input);
    if (user_input > magic_number) {
      puts("You number is bigger!");
    } else if (user_input < magic_number) {
      puts("You number is smaller!");
    } else {
      puts("Yes! You got it!");
      break;
    }
  }

  return 0;
}
```

## 函数和程序结构

### 函数基础

```c
#include <stdio.h>

// 函数定义在调用之前
// x 形式参数
double f(double x) {
  return x * x + x + 1;
}

double g(double x, double y, double z) {
  return x * x + y * y + z * z;
}

int main() {
  puts("HelloWorld");

  // 2.0 实际参数
  double result_f = f(2.0);
  double result_g = g(3.0, 4.0, 5.0);
  printf("result of f: %f\n", result_f);
  printf("result of g: %f\n", result_g);

  return 0;
}
```

### 函数原型

```c
// 函数原型
void EmptyParamList(void);
/**
 * 1. 函数名
 * 2. 函数返回值类型（默认为 int）
 * 3. 函数参数列表：参数类型和参数的顺序，形参名不重要
 */
int Add(int, int);

#include <stdio.h>
int main() {
  puts("");
  EmptyParamList();
  int result = Add(1,2);
  return 0;
}

void EmptyParamList(void) {

}
```

### 变量类型和作用域

```c
#include <stdio.h>

// 文件作用域
int global_var = 0;
void LocalStaticVar(void) {
  // 静态类型变量
  // 1. 全局作用域，内存不会因函数和块退出而销毁
  static int static_var;
  auto int non_static_var;

  printf("static var: %d\n", static_var++);
  printf("non static var: %d\n", non_static_var++);
}

// 函数原型作用域
double Add(double a, double b);
double Sort(int size, int array[size]);

// 函数作用域
int main() {
  // 自动变量
  auto int value = 0;

  // 块作用域
  {
    auto int a = 0;
  }

  LocalStaticVar();
  LocalStaticVar();
  return 0;
}
```

### 函数变长参数

```c
#include <stdio.h>
#include <stdarg.h>
void HandleVarargs(int arg_count, ...) {
  // 1. 定义 va_list 用于获取变长参数
  va_list args;
  // 2. 开始遍历
  va_start(args, arg_count);
  for (int i = 0; i < arg_count; ++i) {
    // 3. 取出对于参数：va_list, type
    int arg = va_arg(args, int);
    printf("%d: %d\n", i, arg);
  }
  // 4. 结束遍历
  va_end(args);
}

int main() {
  HandleVarargs(4, 1, 2, 3, 4);
  return 0;
}
```

### 函数的递归

```c
unsigned int Factorial(unsigned int n) {
  if (n == 0) {
    return 1;
  } else {
    return n * Factorial(n - 1);
  }
}

unsigned int Fibonacci(unsigned int n) {
  if (n == 1 || n == 0) {
    return n;
  } else {
    return Fibonacci(n - 1) + Fibonacci(n - 2);
  }
}

#include <stdio.h>
int main(void) {
  printf("3! : %d\n", Factorial(3));
  printf("5! : %d\n", Factorial(5));
  printf("10! : %d\n", Factorial(10));

  printf("Fibonacci 3! : %d\n", Fibonacci(3));
  printf("Fibonacci 5! : %d\n", Fibonacci(5));
  printf("Fibonacci 10! : %d\n", Fibonacci(10));
}
```

汉罗塔移动过程：

```c
#include <stdio.h>

void hanoi(int n, char src, char tmp, char dst) {
  if (n == 1) {
    printf("n = %d : %c -> %c\n", n, src, dst);
  } else {
    hanoi(n - 1, src, dst, tmp);
    printf("n = %d : %c -> %c\n", n, src, dst);
    hanoi(n - 1, tmp, src, dst);
  }
}

int main(void) {
  hanoi(3, 'A', 'B', 'C');
  return 0;
}
```

## 预处理和宏

### 文件包含

C语言的编译过程：

![image-20240413154335773](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/2024/202404131543959.png)

### 自定义头文件

定义头文件用于导出函数原型：

```c
#ifndef CHAPTER5_INCLUDE_FACTORIAL_H_
#define CHAPTER5_INCLUDE_FACTORIAL_H_

unsigned int Factorial(unsigned int n);

unsigned int FactorialByIteration(unsigned int n);

#endif //CHAPTER5_INCLUDE_FACTORIAL_H_

```

定义源文件用于编写函数实现：

```c
#include "../include/factorial.h"

unsigned int Factorial(unsigned int n) {
  if (n == 0) {
    return 1;
  } else {
    return n * Factorial(n - 1);
  }
}

unsigned int FactorialByIteration(unsigned int n) {
  unsigned int result = 1;
  unsigned int i = n;
  for (; i > 0; i--) {
    result *= i;
  }
  return result;
}
```

引用头文件使用函数：

```c
// 直接查找搜索路径
#include <stdio.h>
// 首先查找当前源文件所在的路径
#include "include/factorial.h"

int main() {
  printf("3! = %d\n", Factorial(3));
  return 0;
}
```

### 宏函数

```c
#include <stdio.h>

#define MAX(a, b) (a) > (b) ? (a) : (b)

// 定义宏时使用 \ 换行
#define IS_HEX_CHARACTER(ch) \
((ch) >='0' && (ch) <= '9') || \
((ch) >='A' && (ch) <= 'F') || \
((ch) >='a' && (ch) <= 'f')

int Max(int a, int b) {
  return a > b ? a : b;
}

int main() {
  int max1 = MAX(1, 3);
  int max2 = MAX(1, MAX(3, 4));
  printf("max1 = %d\n", max1);
  printf("max2 = %d\n", max2);
  int max3 = Max(1, Max(3, 4));
  printf("max3 = %d\n", max3);

  printf("is A a hex character ? %d", IS_HEX_CHARACTER('a'));
  return 0;
}
```

### 条件编译

```c

#define DEBUG
void dump(char *message) {
#ifdef DEBUG
  puts(message);
#endif
}

int main() {
  dump("main start");
  printf("Hello World!");
  dump("main end");

#if __STDC_VERSION__ >= 201112
  puts("C11!");
#elif __STDC_VERSION__ >= 199901
  puts("C99!");
#else
  puts("maybe C90?");
#endif
  return 0;
}
```

## 数组

### 数组基础

```c
#include <stdio.h>
#include <io_utils.h>

#define ARRAY_SIZE 10

// 初始为元素默认值
int global_array[ARRAY_SIZE];

int main() {
  // 仅开辟空间，没有初始化元素，值不确定
  int array[ARRAY_SIZE];
  for (int i = 0; i < ARRAY_SIZE; ++i) {
//    array[i] = i;
    PRINT_INT(array[i]);
  }

  // 显示指定了数组全部元素
  int array_2[ARRAY_SIZE] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
  for (int i = 0; i < ARRAY_SIZE; ++i) {
    PRINT_INT(array_2[i]);
  }

  // 显示指定了前两个元素，后三个元素为默认值
  double array_double[5] = {0.1, 2.3};
  for (int i = 0; i < 5; ++i) {
    PRINT_DOUBLE(array_double[i]);
  }

  // 使用 [2] = 'o' 给指定位置元素赋值，其余元素为默认值
  char array_char[5] = {[2] = 'o', 'l', 'l'};
  for (int i = 0; i < 5; ++i) {
    PRINT_CHAR(array_char[i]);
  }

  return 0;
}
```

### 字符串数组

```c
#include <stdio.h>
#include <io_utils.h>

int main() {
  // C 语言要求字符数组以 \0 结尾
  char string[] = "Hello World";
  for (int i = 0; i < 11; ++i) {
    PRINT_CHAR(string[i]);
  }

  PRINTLNF("%s", string);

  char string_zh[] = "你好，中国";
  wchar_t ws[] = L"你好，中国";

  return 0;
}
```

### 二维数组

```c
#include <stdio.h>
#include <io_utils.h>

// VLA
// 需要数组返回值，可以通过参数传入
void SumOfArray(int rows, int columns, int array[][columns], int result[]) {
  for (int i = 0; i < rows; ++i) {
    for (int j = 0; j < columns; ++j) {
      result[i] += array[i][j];
    }
  }
}

int main() {
  // 列表初始化
  int vehicle_limits[5][2] = {
      {0, 5},
      {1, 6},
      {2, 7},
      {3, 8},
      {4, 9}
  };

  // int[]
  // vehicle_limits[0];
  for (int i = 0; i < 5; ++i) {
    for (int j = 0; j < 2; ++j) {
      vehicle_limits[i][j] = i + j;
    }
  }

  int scores[5][4] = {
      {1, 2, 3, 4},
      {11, 22, 33, 44},
      {111, 222, 333, 444},
      {1111, 2222, 3333, 4444},
      {11111, 22222, 33333, 44444}
  };
  // 需要将数组元素初始化
  int result[5] = {0};
  SumOfArray(5, 4, scores, result);
  PRINT_INT_ARRAY(result, 5);

  return 0;
}
```

### 快速排序

```c
#include <stdio.h>
#include <io_utils.h>
#include <stdlib.h>
#include <time.h>

// 交换数组中两个元素
void SwapElements(int array[], int first, int second) {
  int temp = array[first];
  array[first] = array[second];
  array[second] = temp;
}

// 数组乱序
void ShuffleArray(int array[], int length) {
  srand(time(NULL));

  for (int i = length - 1; i > 0; --i) {
    int random_number = rand() % i;
    SwapElements(array, i, random_number);
  }
}

int Partition(int ARRAY[], int L, int R) {
  int pivot = ARRAY[R];
  int partition = L;
  for (int i = L; i < R; ++i) {
    if (ARRAY[i] < pivot) {
      SwapElements(ARRAY, i, partition);
      partition++;
    }
  }
  SwapElements(ARRAY, R, partition);
  return partition;
}

// 快速排序
void QuickSort(int ARRAY[], int L, int R) {
  if (L >= R) {
    return;
  }
  int partition = Partition(ARRAY, L, R);
  QuickSort(ARRAY, L, partition - 1);
  QuickSort(ARRAY, partition + 1, R);
}

#define PLAYER_COUNT 50
int main() {
  int players[PLAYER_COUNT];
  for (int i = 0; i < PLAYER_COUNT; ++i) {
    players[i] = i;
  }
  PRINT_INT_ARRAY(players, PLAYER_COUNT)
  ShuffleArray(players, PLAYER_COUNT);
  PRINT_INT_ARRAY(players, PLAYER_COUNT)
  // 排序
  QuickSort(players, 0, PLAYER_COUNT - 1);
  PRINT_INT_ARRAY(players, PLAYER_COUNT)
  return 0;
}
```

## 指针

### 指针基础

```c
#include <stdio.h>
#include <io_utils.h>

int main() {
  int a;
  scanf("%d", &a);

  // 指针是一个变量，存储了一个地址
  int *p = &a;
  PRINT_HEX(p);
  PRINT_HEX(&a);

  // 64 位处理器占8字节
  PRINT_INT(sizeof(int *));

  PRINT_INT(*p);
  PRINT_INT(a);

  return 0;
}
```

### 只读指针变量与只读变量指针

```c
int main() {
  int a;
  int b;

  // 指针变量指向 a 的地址，可以改变 a 的值，可以改变 p 指向的地址
  int *p = &a;
  *p = 10;
  p = &b;

  // 只读指针变量，可以改变 a 的值，不可以改变 cp 指向的地址
  int *const cp = &a;
  *cp = 10;
  // cp = &b; ERROR

  // 指针变量指向只读类型，不可以改变 a 的值，可以改变 cp2 指向的地址
  int const *cp2 = &a;
  // &cp2 = 10; ERROR
  cp2 = &b;

  // 只读指针变量指向只读类型，不可以改变 a 的值，不可以改变 cpp 指向的地址
  int const *const cpp = &a;
  // &cpp = 10; ERROR
  // cpp = &b; ERROR

  return 0;
}
```

### 动态内存分配

```c
#include <stdio.h>
#include <stdlib.h>
#include <io_utils.h>

#define PLAYER_COUNT 10

void InitPointer(int **ptr, int length, int defaultValue) {
  // malloc 返回的内存不会清空
  *ptr = malloc(sizeof(int) * length);
  for (int i = 0; i < length; ++i) {
    (*ptr)[i] = defaultValue;
  }
}

int main() {
  int *players;
//  InitPointer(&players, PLAYER_COUNT, 0);
  // calloc 返回的内存会被清空
  players = calloc(PLAYER_COUNT, sizeof(int));
  for (int i = 0; i < PLAYER_COUNT; ++i) {
    PRINT_INT(players[i]);
    players[i] = i;
  }
  PRINT_INT_ARRAY(players, PLAYER_COUNT)

  int *old = players;
  // realloc 追加的内存不会清空
  players = realloc(players, PLAYER_COUNT * 2 * sizeof(int));
  if (players) {
    PRINT_INT_ARRAY(players, PLAYER_COUNT * 2)
  }
  // 释放内存
  free(old);
  free(players);

  return 0;
}
```

## 聚合数据类型

### 结构体

```c
#include <stdio.h>
#include <io_utils.h>
int main() {
  typedef struct Company {
    char *location;
  } Company;

  typedef struct Person {
    char *name;
    int age;
    char *id;
    Company *company;
    struct {
      int extra;
      char *extra_str;
    } extra_value;
    struct Person *partner;
  } Person;

  Company company = {};
  Person person = {.name="bennyhou", 0, "111111111",
      .company = &company, .extra_value = {
          .extra = 1, .extra_str = ""
      }};
  PRINT_INT(person.age);
  person.age = 30;

  PRINT_HEX(&person);
  struct Person *person_ptr = &person;
  PRINT_INT(person_ptr->age);

  PRINT_INT(sizeof(Person));

  struct {
    char *name;
    int gae;
    char *id;
  } anonymous_person;

  Person person_1 = {.name="andy", .age=20};

  return 0;
}
```

### 联合体

```c
#include <stdio.h>
#include <io_utils.h>

typedef union Operand {
  int int_operand;
  double double_operand;
  char *string_operand;
} Operand;
int main() {
  PRINT_INT(sizeof(Operand));

  // 联合体字段使用时互斥
  Operand operand = {.int_operand = 5, .double_operand = 2.0};
  PRINT_INT(operand.int_operand);
  PRINT_DOUBLE(operand.double_operand);

  return 0;
}
```

### 枚举

```c
#include <stdio.h>
#include <io_utils.h>

typedef enum FileFormat {
  PGN, JPEG = 10, BMP = 5, UNKNOWN
} FileFormat;

FileFormat GuessFormat(char *file_path) {
  FILE *file = fopen(file_path, "rb");
  FileFormat file_format = UNKNOWN;
  if (file) {
    char buffer[8] = {0};
    size_t bytes_count = fread(buffer, 1, 8, file);
    if (bytes_count == 8) {
      // bmp : 42 4D
      if (*((short *) buffer) == 0x4D42) {
        file_format = BMP;
      }
    }
    fclose(file);
  }
  return file_format;
}

int main() {
  FileFormat file_format = UNKNOWN;

  PRINT_INT(file_format);

  return 0;
}
```

## 字符串

### 判断字符的类型

```c
#include <io_utils.h>
#include <ctype.h>

int IsDigit(char c) {
  return c >= '0' && c <= '9';
}

int main() {
  PRINT_INT(isdigit('0'));
  PRINT_INT(isspace(' '));
  PRINT_INT(isalpha('a'));
  PRINT_INT(isalnum('f'));
  PRINT_INT(isalnum('1'));
  PRINT_INT(ispunct(','));

  return 0;
}
```

