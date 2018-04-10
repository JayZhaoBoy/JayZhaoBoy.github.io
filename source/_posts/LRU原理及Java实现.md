---
title: LRU原理及Java实现
catalog: true
date: 2018-04-03 15:20:39
subtitle:
header-img: "/img/header_img/code-bg2.jpg"
tags:
- Android
- Java
---

LRU原理及Java实现
===


## 什么是 LRU ？
LRU：Least Recent Used（最近最少使用）然后其他的我就不多介绍了。写这篇文章的背景也是因为今天在知乎看到了一位大佬写的文章，觉得写的特别好，但是根据大佬的解释又有了自己的一些理解。所以在此总结一下，能力有限，或许有不足之处，望请见谅。先献上大佬链接：[LRU 原理和 Redis 实现](https://zhuanlan.zhihu.com/p/34133067?group_id=956610327769559040)这篇文章对 LRU 的介绍十分透彻，以及写出了如何在 Java 和 Redis 中的实现，由于对 Redis 不甚了解，就不班门弄斧了，只对 Java 部分进行探讨。

## Java 代码实现
文中介绍了使用 HashMap 加 双向链表的实现方式，为什么采用这种方式也介绍的十分清楚，我先盗用其中的几张图展示一下：


![map]($res/map.jpg)

![linked]($res/linked.jpg)

所以要不咋说有图有真相呢？
思路是根据 key 通过 hashMap 保证保存和获取的是同一个对象，文中说的是时间 O(1)倒不是很明白。map 中的 value 存储的是 对应的 LRU 存储中槽的位置。而 LRU 的存储双向链表中，h 代表头部，t 代表尾部，链表的长度是一定的，每次默认插入的位置都是在头部，超出范围则淘汰底部。

核心步骤（文中原话）：
1.  save(key)，首先通过 hash 算法，在key space 找到 key，并且尝试把数据存储到LRU空间，如果LRU空间足够，则不需要淘汰旧的内容；如果缓存空间不足，会淘汰掉 tail 指向的内容，并更新队头，存储后返回使用槽下标，更新key space 的value。

2.  get(key)，通过 hash 找到 key，然后通过 value 找到 LRU空间对应的槽，更新LRU指向关系。

原文中的 Java 代码实现：

```
class DLinkedNode {
	int key;
	int value;
	DLinkedNode pre;
	DLinkedNode post;
}
```
**LRU Cache**

```
public class LRUCache {
   
    private Hashtable<Integer, DLinkedNode>
            cache = new Hashtable<Integer, DLinkedNode>();
    private int count;
    private int capacity;
    private DLinkedNode head, tail;

    public LRUCache(int capacity) {
        this.count = 0;
        this.capacity = capacity;

        head = new DLinkedNode();
        head.pre = null;

        tail = new DLinkedNode();
        tail.post = null;

        head.post = tail;
        tail.pre = head;
    }

    public int get(int key) {

        DLinkedNode node = cache.get(key);
        if(node == null){
            return -1; // should raise exception here.
        }

        // move the accessed node to the head;
        this.moveToHead(node);

        return node.value;
    }


    public void set(int key, int value) {
        DLinkedNode node = cache.get(key);

        if(node == null){

            DLinkedNode newNode = new DLinkedNode();
            newNode.key = key;
            newNode.value = value;

            this.cache.put(key, newNode);
            this.addNode(newNode);

            ++count;

            if(count > capacity){
                // pop the tail
                DLinkedNode tail = this.popTail();
                this.cache.remove(tail.key);
                --count;
            }
        }else{
            // update the value.
            node.value = value;
            this.moveToHead(node);
        }
    }
    /**
     * Always add the new node right after head;
     */
    private void addNode(DLinkedNode node){
        node.pre = head;
        node.post = head.post;

        head.post.pre = node;
        head.post = node;
    }

    /**
     * Remove an existing node from the linked list.
     */
    private void removeNode(DLinkedNode node){
        DLinkedNode pre = node.pre;
        DLinkedNode post = node.post;

        pre.post = post;
        post.pre = pre;
    }

    /**
     * Move certain node in between to the head.
     */
    private void moveToHead(DLinkedNode node){
        this.removeNode(node);
        this.addNode(node);
    }

    // pop the current tail.
    private DLinkedNode popTail(){
        DLinkedNode res = tail.pre;
        this.removeNode(res);
        return res;
    }
}
```
但是看完代码我们发现，实现方式并不是文中所说的，map 的 value 存储的是 LRU 中存储的槽的位置，而是链表本身。而且因为文中的 value 用的是 int 值，就变得更具迷惑性。这里的 value 代表的应该是需要存储的数据对象。
——————————————————————————————
（2018.3.30更新）割～原文作者已经做了相应的更改及配图

## 那么问题来了

* 为什么采用 HashMap + 双向链表实现？好处在哪里？
* 为什么 map 中的 value 存储的不是槽的位置而是双向链表的一个节点呢？
* 数组可以实现吗（同队列和列表）
* 用单向链表可以实现么？
* LinkedHashMap可以实现吗？

### 为什么为什么采用 HashMap + 双向链表实现？好处在哪里？
好处先不说，先去分析以下几种方式

### 为什么 map 中的 value 存储的不是槽的位置而是双向链表的一个节点呢？
因为双向链表我们在插入的时候就保证了插入的顺序，而且链表本身虽然是有序的，但不像数组和列表一样是有明确的索引的，然而双向链表的特点：一个节点既是节点本身，也是整个链表，所以 value 也就相当于持有了链表的索引。如果在单独进行索引的存储，还要维护索引的更新和通过索引去查找，显得有些多此一举。

###  列表可以实现吗（同队列和数组）
在程序的世界好像没什么是不可能的，所以答案是肯定的。
代码其实和用链表非常类似，但还是写一下吧，这里暂时就不考虑线程安全的问题了。So：

```

public class TData<T> {
    private String key;
    private T t;

    public TData(String key, T t) {
        this.key = key;
        this.t = t;
    }

    public String getKey() {
        return key;
    }

    public void setKey(String key) {
        this.key = key;
    }

    public T getT() {
        return t;
    }

    public void setT(T t) {
        this.t = t;
    }
}

```

```
public class LRUCacheByList<T> {
    // 泛型 T 表示要存储的数据类型

    private List<TData<T>> mlist;
    private int count = 0;

    // 存储大小
    private int capacity;
    private Map<String, TData<T>> map = new HashMap<>();

    public LRUCacheByList(int capacity) {
        this.capacity = capacity;
        mlist = new ArrayList<>(capacity);
    }

    public void set(String key, T t) {
        TData<T> td = map.get(key);
        if (td != null) {
            // 已经包含则删除原来的，在将其添加到最开始位置
            mlist.remove(td);
            mlist.add(0, td);
        } else {
            td = new TData<>(key, t);
            count = mlist.size();
            if (count < capacity) { //不包含且容量未满，直接添加到最开始位置
                mlist.add(0, td);
                map.put(key, td);
            } else { //不包含且容量已满，先删除末尾位置然后添加到最开始位置
                TData<T> r_td = mlist.get(capacity - 1);
                String k = r_td.getKey();
                map.remove(k);
                mlist.add(0, td);
                map.put(key, td);
            }
        }
    }

    public TData<T> get(String key) {
        TData<T> td = map.get(key);
        if (td != null) {
            mlist.remove(td);
            mlist.add(0,td);
        }
        return td;
    }
}

```
因为list方法封装的原因，操作起来比使用链表的方式简单一些。之所以不采用这种方式是因为我们都知道列表的优点是易于查询，但不宜频繁的进行插入和移除，列表长度短的时候还好。一旦长度变得很大，尤其还要频繁在首部插入数据，内部数组索引要整体后移，对性能的影响还是很大的。

### 单向链表可以实现吗？

因为单向链表只存在下标，所以在链表数据更新的时候按理说是无法进行更新的。比如你获取中间某一位置的对象，你需要将它移除，并将其插入到头部。但是单向链表无法获取上一节点，故无法将其上一节点指向其下一节点。但其实也是有办法的，单向链表也是有序的，只要单独在维护一个 index 的索引，然后每次遍历获得前一节点即可。但这样不仅操作繁琐，亦影响性能。代码这里就不写了。

### LinkedHashMap可以实现吗？
LinkedHashMap虽然有序，但好像无法更新数据结构，所以应是不行的。






