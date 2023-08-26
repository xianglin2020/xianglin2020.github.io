---
title: java 控制流程
date: 2020-09-06 22:49:43
categories: [learn]
tags: [java]
---

# Java 流程控制

三大流程控制语句：顺序、选择、循环

## 选择结构

### `if` 结构

* 判断条件是布尔类型，且是一个范围

```java
Scanner scanner = new Scanner(System.in);
int score = scanner.nextInt();
if (score >= 90) {
  System.out.println("优");
}
if (score >= 80 && score < 90) {
  System.out.println("良");
}
if (score >= 60 && score < 80) {
  System.out.println("中");
}
if (score < 60) {
  System.out.println("不及格");
}
```

#### 多重 `if`

```java
if (score >= 90) {
  System.out.println("优");
} else if (score >= 80) {
  System.out.println("良");
} else if (score >= 60) {
  System.out.println("中");
} else {
  System.out.println("不及格");
}
```

#### 嵌套 `if`

* 将整个 if 块嵌套到另一个 if 块中

```java
		public static void main(String[] args) {
        // 定义两个整型变量并初始化
        int x, y;
        Scanner scanner = new Scanner(System.in);
        x = scanner.nextInt();
        y = scanner.nextInt();
        // 判断 x 和 y 是否相等
        if (x != y) {
            if (x > y) {
                System.out.println("x 大于 y!");
            } else {
                System.out.println("x 小于 y！");
            }
        } else {
            System.out.println("x 和 y 相等！");
        }
    }
```



### `switch`结构

* 判断条件是一个常量

* `switch`结构

  ```java
  switch(condition){
    case 常量表达式 1:
      语句 1;break;
    case 常量表达式 2:
      语句 2;break;
    default:
      语句 3;
  }
  ```

* JDK7.0以后表达式结果可以是 byte、short、int、char 以及 String

## 循环结构

### `while`

```java
while(condition){
  statement;
}
```



### `do-while`

* `do-while`循环最少执行一次
* 循环条件后的分号`;`不能丢

### `for`

```java
public static void main(String[] args) {
        int sum = 0;
        for (int i = 1; i <= 5; i++) {
            sum += i;
        }
        System.out.println("1-5 的累加和是：" + sum);
    }
```

### **三种循环结构的应用场景**

1. 从for循环的结构看，三个表达式会依次被执行到，执行的顺序也是固定的，所以for循环适用于循环次数固定的场景
2. while 循环，只有一个判断条件，结果为布尔值，如果为true就执行循环，为false就不执行。所以while循环适用于不知道循环次数，只知道循环达到某个条件可以执行时使用。
   在循环内，一般需要对循环变量进行改变，否则会发生死循环。
3. do-while循环，与while循环类似，只是判断条件放到了循环最后，不管是否满足条件，循环都会被执行一次。因此，do-while循环适用于不知道循环具体执行次数，只知道满足某个条件继续执行或结束执行，并且循环肯定执行一次时使用。
