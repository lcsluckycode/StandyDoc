# Java 反射

java反射是指程序再运行期间可以拿到一个对象的所有信息。

反射是为了解决再运行期间，对于某个实例一无所知的情况下，如何调用其方法。

## class

JVM为每个加载的 class 创建了对应的 Class 实例，并在实例中保存了该 class  的所有信息，包括类名、包名、父类、实现的接口、所有方法、字段等。因此如果获取了某个 Class 的实例，可以通过这个 Class 实例获取到该实例对应的 class 的所有信息。

这种通过 Class 实例获取 class 信息的方法被称为反射。

获取实例的三种方法。

1. 直接通过一个 class 的静态变量 class 获取

   ```java
   Class cls = String.class;
   ```

   

2. 如果有一个实例变量，可以通过实例变量提供的 getClass() 方法获取

   ```java
   String s = "Hello";
   Class cls = s.getClass();
   ```

   

3. 如果知道一个 class 的完整类名，可以通过静态方法 class.forName() 获取

   ```java
   Class cls = Class.forName("java.lang.String");
   ```

小结：

- JVM为每个加载的`class`及`interface`创建了对应的`Class`实例来保存`class`及`interface`的所有信息；
- 获取一个`class`对应的`Class`实例后，就可以获取该`class`的所有信息；
- 通过Class实例获取`class`信息的方法称为反射（Reflection）；
- JVM总是动态加载`class`，可以在运行期根据条件来控制加载class。

## 访问字段

对于任意一个 Object 实例，只要我们获取了它的 Class ，就可以获取它的一切信息。

Class 类提供了一下集中方法获取字段：

- `Field getField(name)`：根据字段名获取某个 public 的 field （包括父类）
- `Field getDaclareField(name)`：根据字段名获取当前类的某个 field （不包含父类）
- `Field[] getFields()`：获取所有public 的 field （包括父类）
- `Field[] getDeclareFields()`：获取当期类的所有 field（不包括父类）

```java
public class Main {
    public static void main(String[] args) throws Exception {
        Class stdClass = Student.class;
        // 获取public字段"score":
        System.out.println(stdClass.getField("score"));
        // 获取继承的public字段"name":
        System.out.println(stdClass.getField("name"));
        // 获取private字段"grade":
        System.out.println(stdClass.getDeclaredField("grade"));
    }
}

class Student extends Person {
    public int score;
    private int grade;
}

class Person {
    public String name;
}

/*
public int Student.score
public java.lang.String Person.name
private int Student.grade
*/
```

一个`Field`对象包含了一个字段的所有信息：

- `getName()`：返回字段名称，例如，`"name"`；
- `getType()`：返回字段类型，也是一个`Class`实例，例如，`String.class`；
- `getModifiers()`：返回字段的修饰符，它是一个`int`，不同的bit表示不同的含义。

**获取字段的值**

```java
import java.lang.reflect.Field;

public class Main {

    public static void main(String[] args) throws Exception {
        Object p = new Person("Xiao Ming");
        Class c = p.getClass();
        Field f = c.getDeclaredField("name");
        f.setAccessible(true); // 因为该字段为 private ，直接访问会报错，所以需要加上这条指令忽略 private
        Object value = f.get(p);
        System.out.println(value); // "Xiao Ming"
    }
}

class Person {
    private String name;

    public Person(String name) {
        this.name = name;
    }
}
```

**设置字段值**

```java
import java.lang.reflect.Field;

public class Main {

    public static void main(String[] args) throws Exception {
        Person p = new Person("Xiao Ming");
        System.out.println(p.getName()); // "Xiao Ming"
        Class c = p.getClass();
        Field f = c.getDeclaredField("name");
        f.setAccessible(true);
        f.set(p, "Xiao Hong");
        System.out.println(p.getName()); // "Xiao Hong"
    }
}

class Person {
    private String name;

    public Person(String name) {
        this.name = name;
    }

    public String getName() {
        return this.name;
    }
}
```

## 调用方法

`Class`类提供了以下几个方法来获取`Method`：

- `Method getMethod(name, Class...)`：获取某个`public`的`Method`（包括父类）
- `Method getDeclaredMethod(name, Class...)`：获取当前类的某个`Method`（不包括父类）
- `Method[] getMethods()`：获取所有`public`的`Method`（包括父类）
- `Method[] getDeclaredMethods()`：获取当前类的所有`Method`（不包括父类）

```java
public class Main {
    public static void main(String[] args) throws Exception {
        Class stdClass = Student.class;
        // 获取public方法getScore，参数为String:
        System.out.println(stdClass.getMethod("getScore", String.class));
        // 获取继承的public方法getName，无参数:
        System.out.println(stdClass.getMethod("getName"));
        // 获取private方法getGrade，参数为int:
        System.out.println(stdClass.getDeclaredMethod("getGrade", int.class));
    }
}

class Student extends Person {
    public int getScore(String type) {
        return 99;
    }
    private int getGrade(int year) {
        return 1;
    }
}

class Person {
    public String getName() {
        return "Person";
    }
}
/*
Note: Main.java uses unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.
public int Student.getScore(java.lang.String)
public java.lang.String Person.getName()
private int Student.getGrade(int)
*/
```

当获取到一个 Method 对象时，就可以对它进行调用。

```java
String s = "Hello world";
String r = s.substring(6); // "world"

import java.lang.reflect.Method;

public class Main {
    public static void main(String[] args) throws Exception {
        // String对象:
        String s = "Hello world";
        // 获取String substring(int)方法，参数为int:
        Method m = String.class.getMethod("substring", int.class);
        // 在s对象上调用该方法并获取结果:
        String r = (String) m.invoke(s, 6);
        // 打印调用结果:
        System.out.println(r); // "world"
    }
}

```

调用静态方法时， `invoke` 传入的第一个参数为null，因为静态方法的调用不需要实例。

调用非 public 的方法。采用与读取非 public 的字段的方式一致，在调用前，先调用 `method.setAccessible(true);`

## 调用构造函数

通过反射来创建新的实例，可以调用 class 提供的 `newInstance()` 方法

`Person p = Person.class.newInstance()`

但是这个方法只能调用public 无参数构造方法。

为了调用任意的构造方法，Java的反射API提供了Constructor对象，它包含一个构造方法的所有信息，可以创建一个实例。Constructor对象和Method非常类似，不同之处仅在于它是一个构造方法，并且，调用结果总是返回实例：

```java
import java.lang.reflect.Constructor;

public class Main {
    public static void main(String[] args) throws Exception {
        // 获取构造方法Integer(int):
        Constructor cons1 = Integer.class.getConstructor(int.class);
        // 调用构造方法:
        Integer n1 = (Integer) cons1.newInstance(123);
        System.out.println(n1);

        // 获取构造方法Integer(String)
        Constructor cons2 = Integer.class.getConstructor(String.class);
        Integer n2 = (Integer) cons2.newInstance("456");
        System.out.println(n2);
    }
}
```

通过Class实例获取Constructor的方法如下：

- `getConstructor(Class...)`：获取某个`public`的`Constructor`；
- `getDeclaredConstructor(Class...)`：获取某个`Constructor`；
- `getConstructors()`：获取所有`public`的`Constructor`；
- `getDeclaredConstructors()`：获取所有`Constructor`。