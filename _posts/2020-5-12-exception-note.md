---
layout: post
title:  Exception note
date:   2020-5-12 00:00:00 +0800
categories: document
tag: bugs
---

>开篇思考：哪些坏味道
1. 大量的 if else 
2. 对象关系不够清晰、细致
3. 逻辑结构层层嵌套，难阅读


# java.util.ConcurrentModificationException

ArrayList 是 java 实际开发中最常用的数据结构之一了，每隔十行代码必有一个 List 。其实 ArrayList 从字面看
就是 Array 数组结构，起内部就是一个 Object[] element； 
 
  
最近遇到对 List 进行 remove 操作，爆出一个并发修改异常 ConcurrentModificationException，
发现如果使用 Iterator 进行迭代遍历，使用 remove 会导致数组长度发生变化，如果此时继续 remove 会爆出异常。

但是我们使用的 foreach 来进行的循环，为啥还会报 Iterator 封装的错误呢？  
原来，java 中的 foreach 它只是 Iterator 循环的语法糖，编写代码的时候简化了代码，
而编译的时候依然是用 Iterator 实现的。

下面我们一起看下源码：

ArrayList iterator  

```

    private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }

```

上面的源码，通过 debug 会发现进入了 iterator 中 checkForComodification() 方法，
判断 ` modCount != expectedModCount ` ,只要 modcount 和预期值不符，就会抛出异常。
那么 modcount 为什么会不符合？初始值是多少？预期值又应该是多少？  

先看 modCount 是如何解释、赋值的 

```
    AbstractList 下的
    /**
     * The number of times this list has been <i>structurally modified</i>.
     * Structural modifications are those that change the size of the
     * list, or otherwise perturb it in such a fashion that iterations in
     * progress may yield incorrect results.
     * 
     */
    protected transient int modCount = 0;



```

当我们结构化的修改 List ，例如 add ，remove 的时候，就会 modCount++; 而我们的 expectedModCount 
一直等于我们初始化的时候的 modCount ，所以当我们遍历的时候进行 add remove 操作，都会报错。

``` 

    // ArrayList add 操作下的 modCount++
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    // ArrayList remove 操作下的 modCount++
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }

```




    


# 喜欢文章请关注我  
  
> 程序领域  
**点击关注+转发，私信发送【面试】或者【资料】可以收获更多资源**

![公众号](https://torgor.github.io/styles/images/my-public-ma.png)








