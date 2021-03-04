**哈希函数**

通过关键字将元素映射到一个位置（桶），这个位置如果没有产生hash冲突，那么则是它的存储地址。查找时，通过hash函数算出存储的实际位置取出地址即可。

hashmap采用链地址法，即数组加链表的方式。

hashmap主干是一个entry数组，每一个entry包含一个kv对。

组成：链表+数组

**重要字段**

**size**：实际存储的kv对的个数

**threshold**：阈值，table分配内存空间后，threshold一般为capacity*loadFatory，hashmap的扩容机制与它有关。

**loadFactor**：：负载因子，table的填充度。为了减缓hash冲突，默认为0.75

**modCount**：hashmap被改变的次数。

**构造函数**

有四个构造器，如果用户不传入initialCapacity和loadFactor。则使用默认值16，0.75.构造函数中不会创建table数组，只有在执行put操作时才真正构建table数组。

**put时存储的确定流程**

key--》hashcode()--》hashcode--》hash--》h--》indexfor（即h&length-1）--》存储位置

hashcode获取哈希值，hash函数进一步保证散列均匀，indexfor找到存储位置，保证得到的位置范围一定在数组范围内。

**加入entry实体时**

如果size超过阈值或者即将发生hash冲突，则将table扩容到之前数组的两倍，将当前的entry数组中元素全部传输过去，扩容之后的数组长度是之前的两倍。

```
void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);//当size超过临界阈值threshold，并且即将发生哈希冲突时进行扩容
            hash = (null != key) ? hash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }

        createEntry(hash, key, value, bucketIndex);
    }
```

**为什么hashmap的长度一定是2的次幂**

最后元素的地址位置为h&length-1，如果hashmap的长度是2的次幂，那么length-1的低位由一堆1组成。这样在与h进行与运算时，首先可以减少hash冲突的概率，因为length-1的高位不会对结果产生影响，要得到某一个存储位置，只有一种h的组合。如果不为二次幂，那么h的低位部分不再具有唯一性，hash冲突几率会变大。数组长度保持2的次幂，length-1的低位都为1，会使得获得的数组索引index更加均匀。在扩容时，也能尽量保证得到的新的数组索引和老数组索引一致(大大减少了之前已经散列良好的老数组的数据位置重新调换)

**get方法**

```
public V get(Object key) {
　　　　 //如果key为null,则直接去table[0]处去检索即可。
        if (key == null)
            return getForNullKey();
        Entry<K,V> entry = getEntry(key);
        return null == entry ? null : entry.getValue();
 }
 
 final Entry<K,V> getEntry(Object key) {
            
        if (size == 0) {
            return null;
        }
        //通过key的hashcode值计算hash值
        int hash = (key == null) ? 0 : hash(key);
        //indexFor (hash&length-1) 获取最终数组索引，然后遍历链表，通过equals方法比对找出对应记录
        for (Entry<K,V> e = table[indexFor(hash, table.length)];
             e != null;
             e = e.next) {
            Object k;
            if (e.hash == hash && 
                ((k = e.key) == key || (key != null && key.equals(k))))
                return e;
        }
        return null;
    }    
```

**重写equal方法需同时重写hashcode方法**

