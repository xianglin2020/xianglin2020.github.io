---
title: java 运算符
date: 2020-09-06 22:49:34
categories: [ learn ]
tags: [ java ]
---

# Java 运算符

## 表达式

* 表达式由运算符和操作数组成

## 运算符

### 算术运算符

* 算术运算符主要用于进行基本的算术运算，包括加减乘除

  | 算术运算符 | 名称  | 举例           |
    |:-----:|:---:|:-------------|
  |   +   | 加法  | 5+10=15      |
  |   -   | 减法  | 10-5=5       |
  |   *   | 乘法  | 3*6=18       |
  |   /   | 除法  | 10/2=5       |
  |   %   | 取余数 | 3%2=1        |
  |  ++   | 自增1 | int n;n++;   |
  |  --   | 自减1 | int n=4;--n; |

* 自增自减运算符

  | 表达式          | 运算顺序                   | 结果             |
    |--------------|------------------------|----------------|
  | num2=++num1; | num1=num1+1;num2=num1; | num1=2;num2=2; |
  | num2=num1++; | num2=num1;num1=num1+1; | num1=2;num2=1; |
  | num2=--num1; | num1=num1-1;num2=num1; | num1=0;num2=0; |
  | num2=num1--; | num2=num1;num1=num1-1; | num2=1;num1=0; |

### 赋值运算符

* 格式：`变量=表达式;`

* 赋值运算符是从有往左运算！

* 赋值运算符的左边不能为常量

* 复合赋值运算符

  | 运算符 | 表达式 | 计算  | 结果（假设 x=15） |
    | ------ | ------ | ----- | ----------------- |
  | +=     | x+=5   | x=x+5 | 20                |
  | -=     | x-=5   | x=x-5 | 10                |
  | *=     | x*=5   | x=x*5 | 75                |
  | /=     | x/=5   | x=x/5 | 3                 |
  | %=     | x%=5   | x=x%5 | 0                 |

### 关系运算符

* 用于判断两个数据的大小

* 比较的结果是一个布尔值

  | 运算符 |   名称   |
    | :----: | :------: |
  |   >    |   大于   |
  |   <    |   小于   |
  |   >=   | 大于等于 |
  |   <=   | 小于等于 |
  |   ==   |   等于   |
  |   !=   |  不等于  |

* 关于运算符主要用于条件结构或循环结构中

### 逻辑运算符

* 用于连接一个或多个条件，判断这些条件是否成立

* 逻辑运算符的结果是布尔值

  | 名称 |  运算符  |    表达式    |
    | :--: | :------: | :----------: |
  |  与  |  &&或&   |  opt1&&opt2  |
  |  或  | \|\|或\| | opt1\|\|opt2 |
  |  非  |    ！    |     !opt     |

* `&&`和`&`都可以表示逻辑与，但他们是有区别的，共同点是他们两边的条件都成立的时候最终结果才是`true`；不同点是`&&`
  只要是第一个条件不成立为`false`，就不会再去判断第二个条件，最终结果直接为`false`，而`&`判断的是所有的条件；

* `||`和`|`都表示逻辑或，共同点是只要两个判断条件其中有一个成立最终的结果就是`true`，区别是`||`只要满足第一个条件，后面的条件就不再判断，而
  `|`要对所有的条件进行判断。

### 条件运算符

* 简单 `if` 语句的格式：

  ```
  if(condition){
      code
  }
  ```

* `if-else`语句的形式

  ```
  if(true){
      code
  }else{
      anothor code
  }
  ```

* Java中的条件运算符是三目运算符。`condition?expression1:expression2`

### 位运算符

### 运算符的优先级

|             运算符              |       描述        |
|:----------------------------:|:---------------:|
|             `()`             |       圆括号       |
|        `!`、`++`、`--`         |    逻辑非、自增、自减    |
|         `*`、`/`、`%`          |     乘、除、取余      |
|           `+`、`-`            |       加、减       |
|      `<`、`<=`、`>`、`>=`       | 小于、小于等于、大于、大于等于 |
|          `==`、`!=`           |     等于、不等于      |
|             `&&`             |       逻辑与       |
|              `               |                 |`                |             逻辑非             |
| `=`、`+=`、`-=`、`*=`、`%=`、`/=` |  赋值运算符、符合赋值运算符  |
