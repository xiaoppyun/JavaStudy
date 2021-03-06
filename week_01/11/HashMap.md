HashMap在JDK1.8之前的实现方式 数组+链表,但是在JDK1.8后对HashMap进行了底层优化,改为了由 数组+链表+红黑树实现,主要的目的是提高查找效率。

1.继承关系

public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable

2.常量&构造方法 //这两个是限定值 当节点数大于8时会转为红黑树存储 static final int TREEIFY_THRESHOLD = 8; //当节点数小于6时会转为单向链表存储 static final int UNTREEIFY_THRESHOLD = 6; //红黑树最小长度为 64 static final int MIN_TREEIFY_CAPACITY = 64; //HashMap容量初始大小 static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16 //HashMap容量极限 static final int MAXIMUM_CAPACITY = 1 << 30; //负载因子默认大小 static final float DEFAULT_LOAD_FACTOR = 0.75f; //Node是Map.Entry接口的实现类 //在此存储数据的Node数组容量是2次幂 //每一个Node本质都是一个单向链表 transient Node<K,V>[] table; //HashMap大小,它代表HashMap保存的键值对的多少 transient int size; //HashMap被改变的次数 transient int modCount; //下一次HashMap扩容的大小 int threshold; //存储负载因子的常量 final float loadFactor;

//默认的构造函数 public HashMap() { this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted } //指定容量大小 public HashMap(int initialCapacity) { this(initialCapacity, DEFAULT_LOAD_FACTOR); } //指定容量大小和负载因子大小 public HashMap(int initialCapacity, float loadFactor) { //指定的容量大小不可以小于0,否则将抛出IllegalArgumentException异常 if (initialCapacity < 0) throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity); //判定指定的容量大小是否大于HashMap的容量极限 if (initialCapacity > MAXIMUM_CAPACITY) initialCapacity = MAXIMUM_CAPACITY; //指定的负载因子不可以小于0或为Null，若判定成立则抛出IllegalArgumentException异常 if (loadFactor <= 0 || Float.isNaN(loadFactor)) throw new IllegalArgumentException("Illegal load factor: " + loadFactor);

    this.loadFactor = loadFactor;
    // 设置“HashMap阈值”，当HashMap中存储数据的数量达到threshold时，就需要将HashMap的容量加倍。
    this.threshold = tableSizeFor(initialCapacity);
}
//传入一个Map集合,将Map集合中元素Map.Entry全部添加进HashMap实例中
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    //此构造方法主要实现了Map.putAll()
    putMapEntries(m, false);
}
3.Node单向链表的实现 //实现了Map.Entry接口 static class Node<K,V> implements Map.Entry<K,V> { final int hash; final K key; V value; Node<K,V> next; //构造函数 Node(int hash, K key, V value, Node<K,V> next) { this.hash = hash; this.key = key; this.value = value; this.next = next; }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }
    //equals属性对比
    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}