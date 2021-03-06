参考：
1. [iOS单向链表数据结构](https://www.jianshu.com/p/9d017562bfb9)



---



链表是一种物理存储单元上非连续、非顺序的存储结构，数据元素的逻辑顺序是通过链表中的指针链接次序实现的。链表由一系列结点（链表中每一个元素称为结点）组成，结点可以在运行时动态生成。每个结点包括两个部分：一个是存储数据元素的数据域，另一个是存储下一个结点地址的指针域。

相比于线性表顺序结构，操作复杂。由于不必须按顺序存储，链表在插入的时候可以达到O(1)的复杂度，比另一种线性表顺序表快得多，但是查找一个节点或者访问特定编号的节点则需要O(n)的时间，而线性表和顺序表相应的时间复杂度分别是O(logn)和O(1)。如下图：

![单向链表数据结构_1.jpg](https://upload-images.jianshu.io/upload_images/843214-659c8b80c177e99f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



下面处理的全部是单链表
单链表的基本结构：

```c
typedef struct node {
    char *data; 
    struct node *next; 
} node_t;
```
设定一个打印链表的函数:

```c
void list_display(node_t *head)
{
    for (; head; head = head->next)
        printf("%s ", head->data);
    printf("\n");
}
```



### 1.计算一个链表的长度（复杂度O(n)）
算法：定义一个p指针指向头结点，步长为1，遍历链表。
```c
int list_len(node_t *head)
{
    int i; 
    for (i = 0; head; head = head->next, i++); 
    return i; 
}
```

### 2.反转链表（复杂度O(n)）
算法：t遍历链表，q记录t的上一个结点，p是一个临时变量用来缓存t的值。
```c
void reverse(node_t *head)
{
    node_t *p = 0, *q = 0, *t = 0; 
    for (t = head; t; p = t, t = t->next, p->next = q, q = p); 
}
```
### 3.查找倒数第k个元素（尾结点记为倒数第0个）（复杂度O(n)）
算法：2个指针p，q初始化指向头结点。p先跑到k结点处，然后q再开始跑。当p跑到最后跑到尾巴时，q正好到达倒数第k个。
```c
node_t *_kth(node_t *head, int k)
{
    int i = 0; 
    node_t *p = head, *q = head; 
    for (; p && i < k; p = p->next, i++); 
    if (i < k) return 0;
    for (; p->next; p = p->next, q = q->next); 
    return q; 
}
```
### 4.查找中间结点（复杂度O(n)）
算法：设两个初始化指向头结点的指针p，q。p每次前进两个结点，q每次前进一个结点，这样当p到达链表尾巴的时候，q到达了中间。
```c
node_t *middle(node_t *head)
{
    node_t *p, *q; 
    for (p = q = head; p->next; p = p->next, q = q->next){
        p = p->next; 
        if (!(p->next)) break; 
    }
    return q; 
}
```

### 5.逆序打印链表（复杂度O(n)）
算法：使用递归（即让系统使用栈）。
```c
void r_display(node_t *t)
{
    if (t){
        r_display(t->next); 
        printf("%s", t->data);
    }
}
```
### 6.判断一个链表是否有环（复杂度O(n)）
算法：设两个指针p、q，初始化指向头。p以步长2的速度向前跑，q的步长是1。这样，如果链表不存在环，p和q肯定不会相遇。如果存在环，p和q一定会相遇。
```c
int any_ring(node_t *head)
{
    node_t *p, *q; 
    for (p = q = head;p; p = p->next, q = q->next){
        p = p->next; 
        if (!p) break; 
        if (p == q) return 1; //yes
    }
    return 0; //fail find
}
```
### 7.找出链表中环的入口结点（复杂度O(n)）
算法：初始化三个指针p，q，r全部指向head，然后p以2的步长行进，q以1的步长行进。当p和q相遇的时候，发出r指针以1的步长行进，当q和r相遇的结点就是环的入口结点。
```c
node_t *find_entry(node_t *head)
{
    node_t *p, *q, *r; 

    for (p = q = head; p; p = p->next, q = q->next){
        p = p->next; 
        if (!p) break; 
        if (p == q) break; 
    }

    if (!p) return 0; //no ring in list

    for (r = head, q = q->next; q != r; r = r->next, q = q->next); 

    return r; 
}
```
解析:

![单向链表数据结构_2.jpg](https://upload-images.jianshu.io/upload_images/843214-c19215c910b491e5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用俩指针p和q，p扫描的步长为1，q扫描的步长为2。它们的相遇点为图中meet处(在环上)。
假设头指针head到入口点entry之间的距离是K。则当q入环的时候，p已经领先了q为: d = K%n(n为环的周长)。
我们设meet处相对entry的距离(沿行进方向)为x, 则有
(n-d)+x = 2x (p行进的路程是q的两倍)
解得x = n-d
那么当p和q在meet处相遇的时候, 从head处再发出一个步长为1的指针r, 可以知道, r和q会在entry处相遇!

### 8.判断两个链表是否相交（复杂度O(m+n)）
算法：两个指针遍历这两个链表,如果他们的尾结点相同,则必定相交。
```c
int is_intersect(node_t *a, node_t *b)
{
    if (!a || !b) return -1; //a or b is NULL
    for (; a->next; a = a->next); 
    for (; b->next; b = b->next); 
    return a == b?1:0; //return 1 for yes, 0 for no
}
```
### 9.找两个相交的链表的交点
算法：p，q分别遍历链表a，b，假设q先到达NULL，此时从a的头结点发出一个指针t，当p到达NULL时，从b的头结点发出s，当s==t的时候即相交。
```c
node_t *intersect_point(node_t *a, node_t *b)
{
    node_t *p, *q, *k, *t, *s; 
    for (p = a, q = b; p && q; p = p->next, q = q->next); 

    k = (p == 0)?q:p; //k record the pointer not NULL
    t = (p == 0)?b:a; //if p arrive at tail first, t = b ; else p = a
    s = (p == 0)?a:b; 
    for (; k; k = k->next, t = t->next); 
    for (; t != s; t = t->next, s = s->next); 
    return t; 
}
```
### 10.删除结点d（不给头结点）
算法：把下一个结点e的数据拷贝到d结点的数据区，然后删除e。（缺陷：不能删除尾结点）
```c
node_t *delete(node_t *d) 
{
    node_t *e = d->next; 
    d->data = e->data; 
    d->next = e->next; 
}
```
### 11.两个链表右对齐打印
算法：p和q两个指针分别遍历链表a和b，假如q先到达NULL（即a比b长），此时由a头结点发出指针t，打印完整的链表a。同时p继续移动到NULL，并且打印空格。同时还从b头结点发出指针s打印完整的链表b。
```c
void foo(node_t *a, node_t *b)
{
    node_t *p, *q, *k, *t, *s; 
    for (p = a, q = b; p && q; p = p->next, q = q->next); 

    k = p?p:q; 
    t = p?a:b; 
    s = p?b:a; 

    for (; t; printf("%d ", t->data), t = t->next); 
    printf("\n");
    for (; k; printf("  "), k = k->next); 
    for (; s; printf("%d ", s->data), s = s->next); 
}
```

