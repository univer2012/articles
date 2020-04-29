## 单链表

1.`LinkList.h`
```c++
template <class T>
struct Node {
    T data;
    Node<T> *next;
};
template <class T>

class LinkList {
public:
    LinkList( );  //建立只有头结点的空链表
    LinkList(T a[ ], int n);  //建立有n个元素的单链表
    ~LinkList();             //析构函数
    int Length();          //求单链表的长度
    T Get(int i);           //取单链表中第i个结点的元素值
    int Locate(T x);       //求单链表中值为x的元素序号
    void Insert(int i, T x);   //在单链表中第i个位置插入元素值为x的结点
    T Delete(int i);        //在单链表中删除第i个结点
    void PrintList( );           //遍历单链表，按序号依次输出各元素
private:
    Node<T> *first;  //单链表的头指针
    
};
```


2.`LinkList.cpp`
```c++
#include <iostream>
using namespace std;

#include "LinkList.h"

/*
 *前置条件：单链表不存在
 *输    入：无
 *功    能：构建一个单链表
 *输    出：无
 *后置条件：构建一个单链表
 */
template <class T>
LinkList<T>::LinkList()
{
    first=new Node<T>;      //生成头结点
    first->next = NULL ;    //头结点的指针域置为空
}

/*
 *前置条件：单链表不存在
 *输    入：顺序表信息的数组形式a[],单链表长度n
 *功    能：将数组a[]中元素建为长度为n的单链表
 *输    出：无
 *后置条件：构建一个单链表
 */
template <class T>
LinkList<T>::LinkList(T a[],int n) {
#if 0   /* 尾插法建立单链表 */
    first=new Node<T>;   //生成头结点
    Node<T> *r,*s;
    r=first;          //尾指针初始化
    for (int i=0; i<n; i++) {
        s=new Node<T>;s->data=a[i];  //为每个数组元素建立一个结点
        r->next=s;r=s;      //插入到终端结点之后
    }
    r->next=NULL;    //单链表建立完毕，将终端结点的指针域置空
    
#elif 1 /* 头插法建立单链表 */
    first=new Node<T>; first->next=NULL;    //初始化一个空链表
    Node<T> *s;
    for (int i=0; i<n; i++) {
        s=new Node<T>; s->data=a[i];            //为每个数组元素建立一个结点
        s->next=first->next;    first->next = s ;  //将结点 s 插入到头结点之后
    }
#endif
}

/*
 *前置条件：无
 *输    入：无
 *功    能：无
 *输    出：无
 *后置条件：无
 */
template <class T>
LinkList<T>::~LinkList() {
    Node<T> *q;
    while (first != NULL) {
        q=first;
        first=first->next;
        delete q;
    }
}

/*
 *前置条件：单链表存在
 *输    入：查询元素位置i
 *功    能：按位查找位置为i的元素并输出值
 *输    出：查询元素的值
 *后置条件：单链表不变
 */
template <class T>
T LinkList<T>::Get(int i) {
    Node<T> *p;int j;
    p=first->next;  j=1;    //工作指针p 和 累加器 i 初始化
    while (p && j<i) {
        p=p->next;      //各种指针 p 后移
        j++;
    }
    if (!p) throw "p为空";
    else return p->data;
}

/*
 *前置条件：单链表存在
 *输    入：查询元素值x
 *功    能：按值查找值的元素并输出位置
 *输    出：查询元素的位置
 *后置条件：单链表不变
 */
template <class T>
int LinkList<T>::Locate(T x) {
    Node<T> *p;int j;
    p=first->next;j=1;      //工作指针p 和 累加器 i 初始化
#if 0
    if (p && p->next) {
        while (p->data != x) {
            p=p->next;
            j++;
        }
        return j;
    } else throw "p为空";
#endif
    while (p != NULL) {
        if (p->data == x) return j; //查找成功，结束函数并返回序号
        p=p->next;
        j++;
    }
    return 0;   //退出循环表明查找失败
}

/*
 *前置条件：单链表存在
 *输    入：插入元素x,插入位置i
 *功    能：将元素x插入到单链表中位置i处
 *输    出：无
 *后置条件：单链表插入新元素
 */
template <class T>
void LinkList<T>::Insert(int i, T x) {
    Node<T> *p;int j;
    p=first;j=0;        //工作指针 p 硬纸箱头结点
    while (p && j<i-1) {    //查找第 i-1个节点
        p=p->next;          //各种指针 p 后移
        j++;
    }
    if (!p) throw "位置";     //没有找到第 i-1 个结点
    else {
        Node<T> *s;
        s=new Node<T>;
        s->data=x;           //申请一个结点s，其数据域为x
        s->next=p->next;
        p->next=s;           //将结点s插入到结点 p 之后
    }
}

/*
 *前置条件：单链表存在
 *输    入：无
 *功    能：输出单链表长度
 *输    出：单链表长度
 *后置条件：单链表不变
 */
template <class T>
int LinkList<T>::Length() {
    Node<T> *p=first->next; int i=0;    //工作指针p和累加器 i 初始化
    while (p) {
        p=p->next;
        i++;
    }
    return i;       //注意 i 的初始化和返回值之间的关系
}

/*
 *前置条件：单链表存在
 *输    入：要删除元素位置i
 *功    能：删除单链表中位置为i的元素
 *输    出：无
 *后置条件：单链表删除元素
 */
template <class T>
T LinkList<T>::Delete(int i) {
    Node<T> *p; int j;
    p=first;j=0;        //注意工作指针 p 要指向头结点
    while (p && j<i-1) {    //查找第 i-1 个结点
        p=p->next;
        j++;
    }
    if (!p || !p->next) throw "位置"; //结点 p 不存在或 p 的后继结点不存在
    else {
        Node<T> *q;int x;
        q=p->next;x=q->data;        //暂存被删结点
        p->next=q->next;            //摘链
        delete q;
        return x;
    }
}

/*
 *前置条件：单链表存在
 *输    入：无
 *功    能：单链表遍历
 *输    出：输出所有元素
 *后置条件：单链表不变
 */
/* 单链表遍历算法 */
template <class T>
void LinkList<T>::PrintList() {
    Node<T> *p;
    p=first->next;          //工作指针p初始化
    while (p != NULL) {
        cout <<p->data<<endl;
        p=p->next;          //工作指针p后移，注意不能写作p++
    }
}
```


## 双链表
```c++
/*
 编写一个完整的程序，实现双向循环链表的基本操作
 （1）利用尾插法建立一个双向循环链表。
 （2）遍历双向循环链表。
 （3）实现双向循环链表中删除一个指定元素。
 （4）在非递减有序双向循环链表中实现插入元素e仍有序算法。
 （5）判断双向循环链表中元素是否对称若对称返回1否则返回0。
 （6）设元素为正整型,实现算法把所有奇数排列在偶数之前。
 （7）在主函数中设计一个简单的菜单调试上述算法。
  */
#include <iostream>
#include <stdio.h>
#include <stdlib.h>


typedef int elemtype;
typedef struct DuLNode {
    elemtype data;
    struct DuLNode *prior;
    struct DuLNode *next;
}DuLNode, *Dulinklist;
int initlist(Dulinklist &L) {   //初始化双向链表
    L=(Dulinklist)malloc(sizeof(DuLNode));  //表头附加结点
    if (!L) exit(-2);
    L->data=0;
    L->next=L;
    L->prior=L;
    return 1;
}
  //初始化了一个空表


void createlist(Dulinklist &L) {    //为茶法生成双向链表
    Dulinklist p,t;
    t=L;
    printf("尾插法：请输入双向链表结点值，以输入0结束。\n");
    scanf("%d",&t->data);
    while (t->data != 0) {
        p=(Dulinklist)malloc(sizeof(DuLNode));
        if (!p) exit(-2);
        t->next=p;
        p->prior=t;
        scanf("%d",&p->data);
        t=p;
    }
    L->prior=t->prior;
    t->prior->next=L;
}
void shuChuList(Dulinklist L) { //通过链表的遍历来输出链表中的信息
    int i;
    Dulinklist p;
    printf("双向链表为：");
    for (p=L,i=1; p->next != L; p=p->next) {
        printf("%d",p->data);
        i++;
    }
    printf("%d\n",p->data);
    printf("双向链表的长度为%d\n\n",i);
}

Dulinklist GetElem(Dulinklist L,int x) {    //获取双向链表中值为X的结点指针
    Dulinklist t,p;
    t=L;
    if (t->data == x)return t;
    t=t->next;
    for (p=NULL; t != L; t=t->next)
        if (t->data == x) {
            p=t;
            break;
        }
    return p;
}

int shanChu(Dulinklist &L,int x) {  //删除链表L中的值为X的结点
    Dulinklist p;
    int flag;
    flag=0;
    while ((p=GetElem(L, x))) {
        p->prior->next=p->next;
        p->next->prior=p->next;
        free(p);
        flag=1;
    }
    if (flag) return 1;
    else return 0;
}

int duiCheng(Dulinklist L) {    //判断双向循环链表中元素是否对称 ，若对称，返回1，否则返回0
    Dulinklist p,q;
    int flag;
    flag=1;
    p=L;
    q=L->prior;
    while (q->prior != p->next && p->next != q) {
        if (p->data != q->data) {
            flag=0;
            break;
        }
        q=q->prior;
        p=p->next;
    }
    if (p == q) flag=1;
    if (p->data != q->data) flag=0;
    return flag;
}

int paiLie(Dulinklist &L) { //设元素为正整型，实现算法把所有奇数排列在偶数之前
    Dulinklist t,p,q;
    t=q=NULL;
    p=L;
    while (p->next != L) {
        if (p->data%2 == 0) {
            if (p == L) L=p->next;  //如果偶结点为主链头结点头指针后移一位
            p->prior->next=p->next;
            p->next->prior=p->prior;
            if (t) {
                q->next=p;
                p->prior=q;
                q=p;
            } else {
                t=q=p;
            }
        }
        p=p->next;
    }
    if (p->data%2 == 0) {
        p->prior->next=p->next;
        p->next->prior=p->prior;
        if (t) {
            q->next=p;
            p->prior=q;
            q=p;
        } else {
            t=q=p;
        }
    }
    if (!t) return 1;   //判断偶数链是否为空
    else if (L ==q) {
        L=t;
        L->prior=q;
        q->next=L;
    } else {    //奇数链和偶数链都不为空执行链接
        L->prior->next=t;
        t->prior=L->prior;
        q->next=L; L->prior=q;
    }
    return 1;
}

void array(Dulinklist &L) { //按升序排列双向链表
    Dulinklist p,q,t1,t2,k;
    for (p=L; p->next != L; p=p->next) {
        k=p;
        for (q=p->next; q != L; q=q->next)
            if ((k->data)>(q->data)) k=q;
        if (k != p) {
            if (p->next == k || k->prior == p) {    //判断k是p的后续结点
                p->prior->next=k;
                k->prior=p->prior;
                k->next->prior=p;
                p->next=k->next;
                p->prior=k;
                k->next=p;
            } else if (p->prior == k || k->next == p) { //判断p是k的后续结点
                p->next->prior=k;
                k->prior->next=p;
                p->prior=k->prior;
                k->next=p->next;
                p->next=k;
                k->prior=p;
            } else {    //当p和k不相邻
                k->prior->next=p;k->next->prior=p;
                p->prior->next=k;p->next->prior=k;
                t1=k->next;t2=k->prior;
                k->next=p->next;k->prior=p->prior;
                p->next=t1;p->prior=t2;
            }
            if (p ==L) L=p=k;
            else p=k;
        }
    }
}

int intoList(Dulinklist &L,int x) { //在非递减有序双向循环链表中实现插入元素e仍有序算法
    Dulinklist p,t;
    p=L;
    t=(Dulinklist)malloc(sizeof(DuLNode));
    t->data=x;
    while (p->data<x && p->next != L)
        p=p->next;
    if (p->next == L && p->data<x) {  //判断尾结点比X小则将t放到p后
        t->next=p->next;t->prior=p;
        p->next->prior=t;p->next=t;
    } else {
        if (p ==L) L=t; //判断头结点比x小则将头指针指向t
        t->prior=p->prior;t->next=p;
        p->prior->next=t;p->prior=t;
    }
    return 0;
}

int main(int argc, const char * argv[]) {
    // insert code here...
    //    std::cout << "Hello, World!\n";
    Dulinklist La,Lb;
    int menu,c,t;
    initlist(La);
    do {
        printf("1 利用尾插法建立一个双向循环链表\n");
        printf("2 遍历双向循环链表\n");
        printf("3 实现双向循环链表中删除一个指定元素\n");
        printf("4 在非递减有序双向循环链表中实现插入元素e仍有序算法\n");
        printf("5 判断双向循环链表中元素是否对称若对称返回1否则返回0\n");
        printf("6 设元素为正整型,实现算法把所有奇数排列在偶数之前\n");
        printf("0 退出\n");
        printf("\n请输入所选菜单（0-6）: ");
        fflush(stdin);
        menu=7;
        if (scanf("%d",&menu) == EOF) menu=7;
        switch (menu) {
            case 1:
                initlist(La);   //如果初始化成功，则创建该链表。
                printf("建立双向链表L----尾插法建立");
                createlist(La);
                break;
            case 2:
                shuChuList(La);break;
            case 3:printf("请输入需要删除的值X ");
                scanf("%d",&c);
                t=shanChu(La, c);
                if (t) {
                    printf("删除成功删除后的链表为：\n");
                    shuChuList(La);
                } else printf("删除失败,未找到 %d\n",c);
                break;
            case 4:printf("升序排列后为：\n");
                array(La);
                shuChuList(La);
                printf("请输入需要插入的值X ");
                scanf("%d",&c);
                intoList(La, c);
                printf("插入后的链表为：\n");
                shuChuList(La);
                break;
            case 5:shuChuList(La);
                t=duiCheng(La);
                if (t) printf("双向链表对称\n\n");
                else printf("双向链表不对称\n\n");
                break;
            case 6:initlist(Lb);
                t=paiLie(La);
                if (t) {
                    printf("重排后为：\n");
                    shuChuList(La);
                } else printf("重排失败！\n");
                break;
            case 0: exit(0);
                
            default:printf("所输入的菜单值不正确!\n\n");
                break;
        }
    } while (menu);
    
    
    
    return 0;
}
```