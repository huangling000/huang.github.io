# Stack

stack继承vector，并提供五种操作，成为一个能够让对象先进后出的vector：push，pop，peek，empty，search。

link Deque：比stack更完整和一致的先进后出stack，应该考虑被优先使用

`Deque<Integer> stack = new ArrayDeque<Integer>（）；`

五个方法

```java
public E push(E item) {
        addElement(item);

        return item;
    }


/**
     * 移除栈顶元素并返回该元素
     *
     * @return  栈顶元素 (vector的最后一个item).
     * @throws  EmptyStackException  如果栈为空
     */
    public synchronized E pop() {
        E       obj;
        int     len = size();

        obj = peek();
        removeElementAt(len - 1);

        return obj;
    }
 /**
     * 查看栈顶元素但不移除
     *
     * @return  栈顶object (vector的最后一个item).
     * @throws  EmptyStackException  if this stack is empty.
     */
    public synchronized E peek() {
        int     len = size();

        if (len == 0)
            throw new EmptyStackException();
        return elementAt(len - 1);
    }
/**
     * Tests if this stack is empty.
     *
     * @return  <code>true</code> if and only if this stack contains
     *          no items; <code>false</code> otherwise.
     */
    public boolean empty() {
        return size() == 0;
    }

 /**
     * Returns the 1-based position where an object is on this stack.
     * If the object <tt>o</tt> occurs as an item in this stack, this
     * method returns the distance from the top of the stack of the
     * occurrence nearest the top of the stack; the topmost item on the
     * stack is considered to be at distance <tt>1</tt>. The <tt>equals</tt>
     * method is used to compare <tt>o</tt> to the
     * items in this stack.
     *
     * @param   o   the desired object.
     * @return  the 1-based position from the top of the stack where
     *          the object is located; the return value <code>-1</code>
     *          indicates that the object is not on the stack.
     */
    public synchronized int search(Object o) {
        int i = lastIndexOf(o);

        if (i >= 0) {
            return size() - i;
        }
        return -1;
    }

    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = 1224463164541339165L;
}

```



