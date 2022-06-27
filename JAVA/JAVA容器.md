# JAVA容器

**常见的容器框架**

<img src='https://github.com/lcsluckycode/StandyDoc/blob/main/image/JAVA_Collection.png?raw=true'/>

从上图可以看出，Java容器主要分为两大阵营： **Collection** 和 **Map**

**Collection**： 主要是单个元素的集合，又List、Queue、Set三个接口区分不同的集合特征，然后由下面的具体类进行实现响应的功能

**Map**： 键值对存储形式的集合

## 1. List

List 的特点是所有的元素 **可以重复** 。主要分为 **ArrayList** 和 **LinkedList** 两种（均为线程不安全）。 

List 的行为特点和数组几乎完全相同，内部按照放入元素的顺序存放，每个元素都可以通过索引确定自己的位置，索引从 0 开始。

Vector 是一个已经被弃用的类，因为它是 **线程同步** 的，加锁的机制会导致数据访问速度变慢。Stack 是满足 **后进后出** 规则的容器，但是 LinkedList 可以实现所有的栈功能。

> `toArray() // 把List转化为Array`

### 1.1 ArrayList

**动态增长** 的 **数组**。 底层是由数组实现，所以 **随机访问速度快**，**插入删除速度慢**

> ArrayList默认的长度是10，如果我们插入的数据超过了10，ArrayList会不断的自我增长。一般扩容为原本的 1.5 倍
>
> ```java
> boolean add(E e); // 在末尾添加元素
> boolean add(index, e); // 在 index 位置添加元素
> E remove(index); // 删除 index 位置的元素
> boolean remove(E e); // 删除 e 元素
> E get(index); // 获取指定索引的元素
> ```

**源码剖析** [参考](https://www.cnblogs.com/ideal-20/p/14380633.html)

构造方法

```java
/**
 * 带参构造函数，参数为用户指定的初始容量
 */
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        // 参数大于0，创建 initialCapacity 大小的数组
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        // 参数为0，创建空数组（成员中有定义）
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        // 其他情况，直接抛异常
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

/**
 * 默认无参构造函数，初始值为 0
 * 也说明 DEFAULT_CAPACITY = 10 这个容量
 * 不是在构造函数初始化的时候设定的（而是在添加第一个元素的时候）
 */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

/**
 * 构造一个包含指定 collection 的元素的列表
 * 这些元素是按照该 collection 的迭代器返回它们的顺序排列的。
 */
public ArrayList(Collection<? extends E> c) {
    // 将给定的集合转成数组
    elementData = c.toArray();
    // 如果数组长度不为 0
    if ((size = elementData.length) != 0) {
        // elementData 如果不是 Object 类型的数据，返回的就不是 Object 类型的数组
        if (elementData.getClass() != Object[].class)
            // 将不是 Object 类型的 elementData 数组，赋值给一个新的 Object 类型的数组
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // 数组长度为 0 ，用空数组代替
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

扩容核心方法

```java
/**
 * ArrayList 扩容的核心方法
 */
private void grow(int minCapacity) {
    // 将当前元素数组长度定义为 oldCapacity 旧容量
    int oldCapacity = elementData.length;
    // 新容量更新为旧容量的1.5倍
    // oldCapacity >> 1 为按位右移一位，相当于 oldCapacity 除以2的1次幂
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 然后检查新容量是否大于最小需要容量，若还小，就把最小需要容量当作数组的新容量
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    // 再检查新容量是否超出了ArrayList 所定义的最大容量
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        // 若超出，则调用hugeCapacity()
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

### 1.2 LinkedList

**链表** 实现的容器。允许元素为空，**双向链表**

**插入删除速度快**，**查询速度慢**

> ```java
> getFirst(),  element(); // 可返回列表的头，但是不删除，假如列表为空，抛出异常
> peek(); // 与上述一致，但是列表为空返回 null
> removeFirst(), remove(); // 删除并返回列表的头， 列表为空，抛出异常
> pool(); // 与上述一致，但是列表为空返回 null
> ```

构造函数

```java
    //集合元素数量
    transient int size = 0;
    //链表头节点
    transient Node<E> first;
    //链表尾节点
    transient Node<E> last;
    //啥都不干
    public LinkedList() {
    }
    //调用     public boolean addAll(Collection<? extends E> c)  将集合c所有元素插入链表中
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);  
    }
```

### 1.3 使用 Iterator 遍历元素

`Iterator` 本身是一个对象，需要由 `List` 的实例调用 `iterator()` 方法的时候创建。可以对每一种不同的类型实现最高效的访问效率。

```java
import java.util.Iterator;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> list = List.of("apple", "pear", "banana"); //Java 9 的做法
        for (Iterator<String> it = list.iterator(); it.hasNext();) {
            String i = it.next();
            System.out.println(s);
        }
    }
}

//apple
//pear
//banana
```



## 2. Queue

**先进先出** 的数据结构。LinkedList提供了技术支持队列操作，并且实现了Queue的接口，所以一般的Queue都可以采用LinkedList实现。

> ```java
> offer(); // 讲一个元素插入对尾
> peek();  // 不移除的情况下将元素插入队尾，队列为空返回null
> element();  // 不移除的情况下将元素插入队尾，队列为空报错
> poll();  // 移除并返回队头，队列为空返回null
> remove();  // 不移除的情况下将元素插入队尾，队列为空报错
> ```

### 2.1 PriorityQueue

**优先队列**

特点是出队的时候，不是遵循先进先出的原则，而是按照元素的优先级顺序出队。调用remove()或者poll() 的方法，返回的总是优先级最高的元素。

假如需要自定义元素的优先级大小，需要手动实现 Comparable 接口。默认最小的元素为优先级最高的元素。

`Queue<Integer> q = new PriorityQueue<>(new MyComparator);`

### 2.2 Deque

双端队列，两头可以进出

> `addLast(E e)\ offerLast(E e); ` 添加元素到队尾
>
> `E removeFirst()\E pollFirst(); ` 取队首元素且删除
>
> `E getFirst()\E peekFirst(); `取队首元素但是不删除
>
> `addFirst(E e)\offerFirst(E e); ` 添加元素到队首
>
> `E removeLast()\E pollLast(); ` 取队尾元素且删除
>
> `E getLast()\E peekLast();` 取队尾元素但是不删除

## 3. Set

数学意义上的集合，即 set 中的元素不能重复

> 判断是否包含元素
>
> `boolean contains(Object e)`

**HashSet**

底层实现为 **散列表** ，查询速度快，本身无序

相当于只存储 HashMap 的 key，具体细节可以参考 hashMap

**TreeSet**

底层为 **红黑树**，有序

相当于只存储 TreeMap 的 key，具体细节可以参考 TreeMap

## 4. Map

**键值对** 存储的一种结构（**Map\<key, value\>**）

> ```java
> Object put(Object key, Object value); // 放进一个键值对，返回值是被替换的值
> Object remove(Object key); // 删除 key 这个键值对
> void putAll(Map mapping);  // 导入另一个 map 的值
> void clear();
> boolean containsKey(Object key);  // 是否包含某个键
> boolean containsValue(Object value); // 是否包含某个值
> 
> public Set keySet(); // 返回这个Map的所有键的集合，因为Map中键是唯一的，所以返回使用一个set
> public Collection values(); // 返回这个Map的所有值的集合，因为值可能重复，所以返回一个Collection
> public Set entrySet(); //返回一个实现Map.Entry接口对象集合，使用这个方法可以遍历每一条记录。
> ```

重复放入 `key-value` 并不会有任何问题，但是一个 `key` 只能关联一个 `value`，所以重复放入的 `value` 会替换之前的 `value`。

**Map遍历示范**

```java
for (Map.Entry<String, String> pair : map.entrySet()) {
    String key = pair.getKey();
    String value = pair.getValue();
}
```

**HashMap**

查找、删除、插入，key无序。

内部采用的比较方式是 `equals()` 方法，当 key 采用自定义的类的时候，必须要正确覆写 `equals` 方法。同时还需要正确实现 `hashCode()` 方法，以便于能够正确就算出正确的 value 索引。

```java
public class Person {
    String firstName;
    String lastName;
    int age;

    @Override
    int hashCode() {
        int h = 0;
        h = 31 * h + firstName.hashCode();
        h = 31 * h + lastName.hashCode();
        h = 31 * h + age;
        return h;
    }
}
// 上述代码假如 firstName 或者 lastName 为 null 的时候，会抛出空指针异常。为了解决该问题，可以借助Object.hash()解决该问题。
int hashCode() {
    return Objects.hash(firstName, lastName, age);
}
```

内部使用数组作为作为 value 的存储，采用 `hashCode()` 计算 key 直接定位 `Value` 的所在的索引。假设不同的 key 的 hashCode 相同的时候， Map 会在同一个位置采用List 结构存储不同的 <Entry<key ，value>>。

**TreeMap**

遍历，底层红黑树

由于TreeMap保证了 key 的有序性存储，所以在进行自定义类作为 key 的存储时，必须正确实现 `Comparable` 接口。如果该类没有实现 `Comparable` 接口，就必须在创建 `TreeMap` 的时候同时指定一个自定义的比较算法。

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        // 实现自定义的 Comparator 算法
        Map<Person, Integer> map = new TreeMap<>(new Comparator<Person>() {
            public int compare(Person p1, Person p2) {
                return p1.name.compareTo(p2.name);
            }
        });
        map.put(new Person("Tom"), 1);
        map.put(new Person("Bob"), 2);
        map.put(new Person("Lily"), 3);
        for (Person key : map.keySet()) {
            System.out.println(key);
        }
        // {Person: Bob}, {Person: Lily}, {Person: Tom}
        System.out.println(map.get(new Person("Bob"))); // 2
    }
}

class Person {
    public String name;
    Person(String name) {
        this.name = name;
    }
    public String toString() {
        return "{Person: " + name + "}";
    }
}
```

