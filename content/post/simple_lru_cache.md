---
title: "缓存 (番外)- 利用 HashMap 和链表实现 LRU_CACHE"
date: 2021-08-24
tags: ["缓存", "算法"]
description : "该篇文章主要是讲述了怎么利用 HashMap 和链表实现 LRU_CACHE"
---

学习缓存淘汰策略时, 有种策略为 LRU。 在学习这个算法中，做了一些笔记。 这个不是官方实现，是从 leetCode 中选了一种比较经典的算法来帮助我看懂原理和自己实现方式。

## 实现一： 使用 Java LinkedHashMap 实现
代码如下：
```java
public class LRUCacheDemo {
    private LinkedHashMap<Integer, Integer> map;

    private final int CAPACITY;

    public LRUCacheDemo(int capacity) {
        CAPACITY = capacity;

        map = new LinkedHashMap<Integer, Integer>(capacity, 0.75f, true) {
            @Override
            protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
                return size() > CAPACITY;
            }
        };
    }

    public int get(int key) {
        return map.getOrDefault(key, -1);
    }

    public void set(int key, int value) {
        map.put(key, value);
    }
}
```
这里使用了 LinkedHashMap 中的 removeEldestEntry 方法，该方法会删除年纪最老的节点，这个年纪最老的节点对比着 JVM 中的 Old 区一起来理解，即经常使用的会被删除，新增就会产生新的对象，而不会被使用的就会一直增长，就会为成为年龄最长的节点。

这是一种投机的做法。如果是面试，这可能不是面试官所需要的答案。那么我们换一种实现。

## 链表 + HashMap 实现
通过上述的代码，我们了解到了，当 cache 中的数据到达容量数时， 会删除年龄最大的节点，不断使用的节点会一直不断产生新的节点。那么我们就用一个链表来表示这个过程， 初始链表中设有头结点和尾结点。用有容量大小的 HashMap 作为容量的限定和链表中节点的记录。节点中有 key，value，上一个节点，下一个节点。 链表和 HashMap 整个组成了一个 cache, 整体如下图所示。

![get_cache](/images/post/lru/864E454A-7135-418a-AB12-12371D5FFD20.png)

获取的数据的时候，从 map 中获取相应的节点，然后删除原有的节点，新建一个节点放入链表中。

节点删除数据流程如下图所示。

![del_node](/images/post/lru/Image_1.png)

删除的大体流程是：现将要删除的节点的 next 节点放入删除节点的 prev 节点的 next 中，将删除节点的 prev 中的值给到删除节点 next 节点中的 prev 中。

节点新增数据流程下图所示：

![add_node](/images/post/lru/BD947AEF-8225-43d9-8B89-53CE5F8087F3.png)

新增的大体流程是：在 head 后面新增一个节点，将 head next 节点放入到 node 的 next 中，将 head 放入 node 的 prev 中， 将 node 节点放入 head next 节点中的 prev 中， 将节点放入 head 节点的 next 中。

这里变完成了节点的新增和删除操作。接下就要往 cache 中 set 数据了。大体流程图如下：

![set_cache](/images/post/lru/Image_3.png)

代码如下：
```java
public class Node {
    private int key;
    private int value;
    private Node prev;
    private Node next;

    Node(int key, int val) {
        this.key = key;
        this.value = val;
    }
}
```


大体流程是：
1. 先判断 map 中是否当前 key， 如果 key 存在就获取当前 node， 并删除链表中相应的 node，修改 node 中 value 的值，并将新的 node 插入到链表中 。
2. 如果不存在当前 key，继续判断是否达到 map 中的容量了，如果 map 中容量满了，则删除 map 中 tail 节点的 prev 节点中 key-node 值，删除链表中 tail.prev 节点， 写入 map 新的 key-node 值， 链表新增新的 node。 如果没有 map 的容量没有满， 则获取 map node 的值，删除链表中原有节点，新增节点， 将新的 key-node 的值写入到 map 中，数量累加。 整体代码如下:

```java
class LRUCache {
    class Node {
        int key, value;
        Node prev, next;

        public Node(int key, int value) {
            this.key = key;
            this.value = value;
        }

    }


    private int size, capacity;
    private Node head, tail;
    private HashMap<Integer, Node> cache;

    public LRUCache(int capacity) {
        head = new Node(0, 0);
        tail = new Node(0, 0);
        this.head.next = tail;
        this.tail.prev = head;
        this.head.prev = null;
        this.tail.next = null;

        this.capacity = capacity;
        size = 0;
        int mapCapacity = (int)(capacity /0.75f + 1);
        cache = new HashMap<>(mapCapacity);
    }


    private void remove(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private void add(Node node) {
        node.prev = head;
        node.next = head.next;

        head.next.prev = node;
        head.next = node;
    }


    public int get(int key) {
       if (cache.containsKey(key)) {
          Node node = cache.get(key);
          remove(node);
          add(node);
          return node.value;
       }

        return -1;
    }

    public void put(int key, int value) {
        Node node;
        if (cache.containsKey(key)) {
            node = cache.get(key);
            remove(node);
            node.value = value;
        } else {
            node = new Node(key, value);

            if (capacity > size) {
                size++;
            } else {
                cache.remove(tail.prev.key);
                remove(tail.prev);
            }

            cache.put(key, node);
        }

        add(node);
    }
}
```








