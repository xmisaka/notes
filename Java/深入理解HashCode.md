> equals()和hashCode()都是Object类中的方法，其中equals()方法我们经常用到，比如String对象的比较str1.equals(str2)。两个方法本来没有任何耦合关系。  
但是把它和hashCode()方法放在一起总结是因为Map集合的实现让这两个方法产生了某种程度上的耦合。

## equals()方法
java散列容器的key都是复杂类型，比较两个key是否相等时，是先通过==比较，如果返回false还要用equals()判断，为什么要这样比较呢？   
在Java中，对于原始数据类型，== 比较的是他们的值，对于复合数据类型(对象)，== 比较的是他们在内存中的存放地址。也就是说如果两个对象的引用指向的是堆上同一个对象时，这两者用==比较才会返回true。  
但是我们在大多数情况下对于散列表中的key相等的条件没有严苛到地址都要一样的地步，只要属性一样就可以认为他们相等，所以==返回false不代表两个key不一样，还要用equals()进一步比较。 比如说String对象的相等判断：

```java
public static void main(String[] args) {
    String str1 = new String("Hello world");
    String str2 = new String("Hello world");
    System.out.println(str1 == str2);        // false
    System.out.println(str1.equals(str2));   // true
}
```
实际情况下我们是可以根据str2来找到散列表中key为str1的元素的，如果只使用==比较来判断key是否匹配是不可行的，因为他们在堆上的地址并不一样，所以得使用equals()来进一步比较。   
```java
public boolean equals(Object obj) {
    return (this == obj);
}
```
之所以我们可以直接使用String的equals()，是因为String类重写了equals()，具体实现就是比较两个字符串对象底层的char数组存放的字符是否完全一样。  
所以在我们要比较自定义的类的对象是否相等的时候，我们要根据自己的需求主动重写equals()，重写equals()也是我们能够把自定义的类作为散列容器中key的类型的先决条件。

## equals() 和 hashCode()
> Effective Java 3rd edition  
  Item 11 : Always override hashCode when you override equals

为什么要这样呢？首先从HashMap的源码我们知道，当从map中找一个指定的key时，我们首先是根据这个key的hashCode()的返回值经过处理后生成的hash值找到对应的桶，再遍历这个桶找到相等的key。  
从以上过程可以提炼出非常重要的一点，相等的对象hashCode()返回值一定要相等，因为如果hashCode()返回值不相等的话，计算出来的hash值很可能是不相等的，这会直接导致相同的key可能放入不同的桶中，这是不被允许的。  
所以如果我们要用自定义类对象做散列表的key时，在重写了equals()后，还要重写hashCode()来保证equals()返回为true的情况下两个对象hashCode()的返回值也相等。

```java
public class Test {
    public static void main(String[] args) {
        HashMap<People, Integer> map = new HashMap<>();
        // 生成两个相等的对象
        People p1 = new People(11, "Jack");
        People p2 = new People(11, "Jack");

        map.put(p1, 1);
        // 当没重写hashCode()时输出的是null
        // 重写时输出的是 1
        System.out.println(map.get(p2));
    }
}   

// 自定义People类
class People {
    int age;
    String name;

    public People(int age, String name) {
        this.age = age;
        this.name = name;
    }

    // 我们认为只要两个People对象age和name相等则这两个对象相等
    @Override
    public boolean equals(Object obj) {
        return ((People) obj).age == age && ((People) obj).name.equals(name);
    }

    // equals()返回为true时，两个对象的hashCode()返回值也是一样的
    @Override
    public int hashCode() {
        return age * name.hashCode();
    }
}
```
## 深入理解hashcode
equals()在Object中默认的实现其实也是使用==比较：code主要是set集合使用，是用于判断对象是否"可能"相等的快捷办法，以解决大集合的问题。  

举例来说，如果一个一万个元素的集合加入一个元素，如果是一个新元素，那么必须要equal一万次才能加入。  
 
所以采用hashcode，hashcode的思路是如果equal，则hashcode一定要相等，反过来则不一定；  
所以如果hashcode不相等，那么一定不equal，这跟md5的hash来判别密码是一个道理。    

hashcode用64位整数，这样可以建立一个索引，新加入元素，先判断这个新元素的hashcode是否存在，如果不存在，肯定不相等，加入set中；  
如果存在，则与已有的hashcode的若干个元素比较，这样大大简化了set的equal操作。  

hash code跟内存没有关系，只不过是Object的默认hashCode()方法会返回一个内存编号，因为这样一定满足hashCode()方法的要求。

hashCode()方法要求：

> 当对象状态未改变，那么多次调用返回的值必须相等  
两个对象equal，那么对象调用返回的值必须相等

你说的那个人的常规思维是认为hash code跟内存相关联，实际上不是，你可以理解为一个数字标识当前对象状态.



