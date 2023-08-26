---
title: java 方法
date: 2020-09-06 22:50:10
categories: [learn]
tags: [java]
hidden: true
---

# 面试题

## JVM 内存组成

![image-20201126213950104](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202011/213950.png)

## 垃圾回收算法



## Java 的内存泄漏

* 静态集合类
* 各种连接

## 实现深拷贝

```java
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;

public class DeepCopyDemo {
    public static void main(String[] args) throws CloneNotSupportedException {
        Dancer dancer = new Dancer();
        Dancer partner = new Dancer();
        partner.setName("Partner");
        dancer.setName("Dancer");
        dancer.setPartner(partner);
        System.out.println("-----原对象-----");
        System.out.println("dancer = " + dancer);
        System.out.println("partner = " + partner);
        System.out.println("-----拷贝后-----");
        Dancer newDancer = (Dancer) dancer.clone();
        System.out.println("newDancer = " + newDancer);
        System.out.println("newPartner = " + newDancer.getPartner());
    }
}

class Dancer implements Cloneable, Serializable {
    private String name;

    private Dancer partner;

    public void setName(String name) {
        this.name = name;
    }

    public void setPartner(Dancer partner) {
        this.partner = partner;
    }

    public String getName() {
        return this.name;
    }

    public Dancer getPartner() {
        return this.partner;
    }

    @Override
    public Object clone() throws CloneNotSupportedException {
        try (ByteArrayOutputStream bos = new ByteArrayOutputStream();
                ObjectOutputStream oos = new ObjectOutputStream(bos);) {
            oos.writeObject(this);
            try (ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
                    ObjectInputStream ois = new ObjectInputStream(bis)) {
                return ois.readObject();
            } catch (Exception exception) {

            }
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }

        return null;
    }
}

/*
----原对象-----
dancer = Dancer@2f0e140b
partner = Dancer@7440e464
-----拷贝后-----
newDancer = Dancer@1d56ce6a
newPartner = Dancer@5197848c
*/
```

