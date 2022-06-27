# Java从入门到放弃（二） 面向对象

**注意：**Java源文件可以包含多个类的定义，但是只能定义一个public类，而且这个public类名必须与文件名一致。如果定义了多个public类，必须拆到多个Java源文件中。

## 1. 构造方法

创造实例的时候，实际上是通过构造方法来初始化实例。

```java
class Persion {
	pravite String name;
	private int age;
	
	public Persion(String name, int age) {
		this.name = name;
		this.age = age;
	}
}
```

值得注意的是，假如定义了一个构造函数，那么编译器就不会默认创建构造函数。

没有在构造方法中初始化字段时，引用类型的字段默认是`null`，数值类型的字段用默认值，`int`类型默认值是`0`，布尔类型默认值是`false`。

## 2. 函数重载

方法名相同，但是各自参数不同，称为方法重载（Overload）

```java
class Hello {
    public void hello() {
        System.out.println("Hello, world!");
    }

    public void hello(String name) {
        System.out.println("Hello, " + name + "!");
    }

    public void hello(String name, int age) {
        if (age < 18) {
            System.out.println("Hi, " + name + "!");
        } else {
            System.out.println("Hello, " + name + "!");
        }
    }
}
```

## 3. 继承

类的继承使用 `extends` 这个关键字，而且一个类**有且只有一个父类**。

**子类自动获得了父类的所有字段**，**严禁定义与父类重名的字段**。

子类**不能访问**父类 `private` 字段或者 `private` 方法。但是**可以访问** `protected` 修饰的字段。

`super` 关键字表示父类（超类）。子类调用父类的字段时，可以用 `super.fieldname`。

```java
class Student extends Person {
    public String hello() {
        return "Hello, " + super.name;
    }
}
```

子类**不会继承**任何父类的构造方法。子类默认的构造方法是编译器**自动生成**的，不是继承的。

## 4. 多态

假如继承关系中，子类定义了一个和父类方法签名完全相同的方法，称为覆写（Override）。这个与重载（Overload）不一样。

```java
class Person {
    public void speak() {
        System.out.println("Person.speak");
    }
}

class Student extends Person {
    @Override  //最好显式指明
    public void speak() {
        System.out.println("Student.speak");
    }
}
```

## 5. 抽象类 和 接口

**抽象类**

含有抽象方法的类为抽象类。

把一个方法声明为`abstract`，表示它是一个抽象方法，本身没有实现任何方法语句。因为这个抽象方法本身是无法执行的，所以，`Person`类也无法被实例化。编译器会告诉我们，无法编译`Person`类，因为它包含抽象方法。

必须把`Person`类本身也声明为`abstract`，才能正确编译它：

```java
abstract class Persion {
    public abstract void run();
}
```

子类必须实现父类的抽象方法。

小结

- 通过`abstract`定义的方法是抽象方法，它只有定义，没有实现。抽象方法定义了子类必须实现的接口规范；
- 定义了抽象方法的class必须被定义为抽象类，从抽象类继承的子类必须实现抽象方法；
- 如果不实现抽象方法，则该子类仍是一个抽象类；
- 面向抽象编程使得调用者只关心抽象方法的定义，不关心子类的具体实现。

**接口**

如果一个抽象类没有字段，所有的方法都是抽象方法，那么这个类可以改写成接口。

```java
abstract class Person {
    public abstract void run();
    public abstract String getName();
}

interface Person {
    void run();
    String getName();
}
```

具体类实现接口时需要 `implement` 关键字。一个类可以实现多个接口。

在接口中，可以定义 `default` 方法。实现类可以不必覆写 `default` 方法。

## 6. 作用域

- **public** 

  定义为`public`的`class`、`interface`可以被其他任何类访问

- **private**

  定义为`private`的`field`、`method`无法被其他类访问。确切地说，`private`访问权限被限定在`class`的内部，而且与方法声明顺序无关。

- **protected** 

  `protected`作用于继承关系。定义为`protected`的字段和方法可以被子类访问，以及子类的子类。

- **package**

  假如没有上述三种关键字修饰。一个类允许访问同一个`package`的没有`public`、`private`修饰的`class`，以及没有`public`、`protected`、`private`修饰的字段和方法。

## 7. classpath 和 jar

`classpath` 时JVM用到的一个环境变量。用来指示JVM如何搜索class。

我们强烈*不推荐*在系统环境变量中设置`classpath`，那样会污染整个系统环境。在启动JVM时设置`classpath`才是推荐的做法。实际上就是给`java`命令传入`-classpath`或`-cp`参数：

```
java -classpath .;C:\work\project1\bin;C:\shared abc.xyz.Hello
```

或者使用`-cp`的简写：

```
java -cp .;C:\work\project1\bin;C:\shared abc.xyz.Hello
```

没有设置系统环境变量，也没有传入`-cp`参数，那么JVM默认的`classpath`为`.`，即当前目录：

```
java abc.xyz.Hello
```

上述命令告诉JVM只在当前目录搜索`Hello.class`。

 **不要把任何Java核心库添加到classpath中！JVM根本不依赖classpath加载核心库！**

`jar`

如果有很多`.class`文件，散落在各层目录中，肯定不便于管理。如果能把目录打一个包，变成一个文件，就方便多了。

jar包就是用来干这个事的，它可以把`package`组织的目录层级，以及各个目录下的所有文件（包括`.class`文件和其他文件）都打成一个jar文件，这样一来，无论是备份，还是发给客户，就简单多了。

jar包实际上就是一个zip格式的压缩文件，而jar包相当于目录。如果我们要执行一个jar包的`class`，就可以把jar包放到`classpath`中：

```
java -cp ./hello.jar abc.xyz.Hello
```

这样JVM会自动在`hello.jar`文件里去搜索某个类。

参考借鉴：廖雪峰的官方网站