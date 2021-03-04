# Vector

Vector实现了一个可增长的对象。

Vector维护一个capacity与capacityIncreaments。capacity至少为vector的size（），但通常更大因为在元素添加时，Vector的存储通常以capacityIncrement的大小增加。在添加大量元素前，应用程序可以先提高vector的容量，这样减少了为了增加容量重新分配内存的次数。

对于迭代器来说，这个类的iterator（） iterator 与listiterator（int） listiterator方法是快速失败的。即在创建该类的迭代器后，若该vector发生了变化，那么迭代器将会抛出ConcurrentModificationException。因此在发生并发修改时，迭代器是fail-fast的。The {@link Enumeration Enumerations} returned by the {@link #elements() elements} method are <em>not</em> fail-fast.

从java1.2开始，这个类被改进为实现List接口，成为java集合框架中的一员。与新的集合实现不同，vector是同步的，如果不是需要线程安全，那么应该使用ArrayList代替Vector。

serialVersionUID的作用



源码

```java
package java.util;

import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.StreamCorruptedException;
import java.util.function.Consumer;
import java.util.function.Predicate;
import java.util.function.UnaryOperator;

public class Vector<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
 	/**
     * vector的元素存入数组中，vector的容量（capacity）则是array的长度，至少能够包含vector的所有元素。vector中最后一个元素后的任何元素都为空。
     * <p>Any array elements following the last element in the Vector are null.
     *
     * @serial
     */

 	protected Object[] elementData;
	
    /**
     * vector中有效的元素个数
     * {@code elementData[0]} 到{@code elementData[elementCount-1]} 中的元素are the actual items.
     *
     * @serial
     */
    protected int elementCount;

    
 	/**
     *当vector的size比容量要大时，vector的capacity将会自动增加。如果容量的增量比0小或和0相等，vector的容量将会double增长。 
     *
     * @serial
     */
    protected int capacityIncrement;
	/** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = -2767605614048989439L;

	/**
     * 构造一个具有初始capacity and capacity increment的vector
     * @param   initialCapacity     the initial capacity of the vector
     * @param   capacityIncrement   vector满时capacity的增量
     * @throws IllegalArgumentException 如果初始capacity是负数
     */
    public Vector(int initialCapacity, int capacityIncrement) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
        this.capacityIncrement = capacityIncrement;
    }
    
 	public Vector(int initialCapacity) {
        this(initialCapacity, 0);
    }
	/**
     * 构造一个空的vector，vector的数据数组大小为10，capacity increment为0.
     */
    public Vector() {
        this(10);
    }

	/**
     * 构造一个有指定集合元素的vector, 按照它们的集合迭代器返回的顺序
     * @param c 一个集合，它的元素即将放入向量
     * @throws NullPointerException if 如果指定集合为null
     * @since   1.2
     */
    public Vector(Collection<? extends E> c) {
        Object[] a = c.toArray();
        elementCount = a.length;
        if (c.getClass() == ArrayList.class) {
            elementData = a;//此处应为ArrayList的toArray函数实质仍然是Arrays的copyOf函数
        } else {
            elementData = Arrays.copyOf(a, elementCount, Object[].class);
        }
    }

 	/**
     * 把vector中的元素复制到anArray中，vector中索引为k的元素被复制给anArray中的k个元素。 
     *
     * @param  anArray 
     * @throws NullPointerException 给的array为null
     * @throws IndexOutOfBoundsException 如果array无法容纳vector中的元素
     * @throws ArrayStoreException 如果vector中的某个组件不是可以存储在array中的运行时类型
     * @see #toArray(Object[])
     */
    public synchronized void copyInto(Object[] anArray) {
        System.arraycopy(elementData, 0, anArray, 0, elementCount);
    }

 	/**
     * 使vector的容量与当前size相同。如果capacity大于当前size，那么用一个小的数组代替原始保存在变量elementData里的数组，使capacity与size一样大。这个操作可以最小化vector的存储。 
     */
    public synchronized void trimToSize() {
        modCount++;//被修改次数+1
        int oldCapacity = elementData.length;
        if (elementCount < oldCapacity) {
            elementData = Arrays.copyOf(elementData, elementCount);
        }
    }

    
	/**
     * Increases the capacity of this vector, if necessary, to ensure
     * that it can hold at least the number of components specified by
     * the minimum capacity argument.
     *
     * <p>If the current capacity of this vector is less than
     * {@code minCapacity}, then its capacity is increased by replacing its
     * internal data array, kept in the field {@code elementData}, with a
     * larger one.  The size of the new data array will be the old size plus
     * {@code capacityIncrement}, unless the value of
     * {@code capacityIncrement} is less than or equal to zero, in which case
     * the new capacity will be twice the old capacity; but if this new size
     * is still smaller than {@code minCapacity}, then the new capacity will
     * be {@code minCapacity}.
     *
     * @param minCapacity the desired minimum capacity
     */
    public synchronized void ensureCapacity(int minCapacity) {
        if (minCapacity > 0) {
            modCount++;//当前集合被修改次数+1
            ensureCapacityHelper(minCapacity);
        }
    }



```

