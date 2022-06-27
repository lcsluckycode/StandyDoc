# Java从入门到放弃（一） 简介&基础

## 1. 版本：

Java SE： Standard Edition（标准版，包含标准的JVM和标准库）

Java EE： Enterprise Edition（企业版，再上面的基础上增加了大量的API和库，以方便开发Web应用、数据库、消息服务等）

Java ME： Micro Edition（嵌入式设备使用）

![JAVA版本之间的关系]()

## 2. JDK 与 JRE

- JDK：Java Development Kit （编译Java源码成Java字节码）
- JRE： Java Runtime Environment （运行Java字节码的虚拟机）

![JDK与JRE]()

- JSR规范：Java Specification Request
- JCP组织： Java Community Process

JSR是一系列的规范，从JVM的内存模型到Web程序接口，全部都标准化了。而负责审核JSR的组织就是JCP。

一个JSR规范发布时，为了让大家有个参考，还要同时发布一个“参考实现”，以及一个“兼容性测试套件”：

- RI：Reference Implementation
- TCK：Technology Compatibility Kit

## 3. 某些知识点

### 1. Java控制台输入

Scanner

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner sca = new Scanner(System.in); //创建Scanner对象
        String name = sca.nextLine(); // 读取一行的输入并获取字符串
        int age = sca.nextInt(); // 获取一行的输入获取整数
    }
}
```

### 2. switch

Java的Switch可以使用，而C++的不能是字符串。

### 3. 多维数组

```java
int[][] ar = new int[10][10];
int[][] ns = {
	{1, 2, 3},
	{4, 5, 6},
	{7, 8, 9}
};

// ns.length = 3;
// ns[0].length = 3;
```

