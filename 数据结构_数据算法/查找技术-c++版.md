# 一、线性表的查找技术

### 1.单链表的顺序查找
### 2.顺序表的顺序查找
### 3.折半查找递归
### 4.折半查找非递归

# 二、树表的查找技术
### 1.二叉排序树查询

# 三、散列表的查找技术
### 1.闭散列表查找HashSearch
### 2.拉链法查找


# 一、线性表的查找技术

### 1.单链表的顺序查找

1.`LinkSearch.h`
```c++
#ifndef ____________LinkSearch__
#define ____________LinkSearch__

#include <stdio.h>
struct Node
{
    int data;
    Node *next;
};

class LinkSearch
{
public:
    void LinkList(int a[ ], int n);    //建立有n个元素的单链表
    void PrintList( );            //遍历单链表，按序号依次输出各元素
    int SeqSearch2(Node *first, int k);//查找单链表中是否存在元素k
    Node *GetFirst( );
private:
    Node *first;              //单链表的头指针
};
#endif /* defined(____________LinkSearch__) */
```
2.`LinkSearch.cpp`
```c++
//LinkSearch.cpp
#include "LinkSearch.h"
#include <iostream>
using namespace std;

/*
 * 前置条件：单链表不存在
 * 输    入：顺序表信息的数组形式a[],单链表长度n
 * 功    能：将数组a[]中元素建为长度为n的单链表
 * 输    出：无
 * 后置条件：构建一个单链表
 */
void LinkSearch::LinkList(int a[],int n)
{
    first=new Node;                   //生成头结点
    Node *r,*s;
    r=first;                          //尾指针初始化
    for (int i=0; i<n; i++)
    {
        s=new Node; s->data=a[i];    //为每个数组元素建立一个结点
        r->next=s; r=s;              //插入到终端结点之后
    }
    r->next=NULL;    //单链表建立完毕，将终端结点的指针域置空
}

/*
 * 前置条件：单链表存在
 * 输    入：无
 * 功    能：单链表遍历
 * 输    出：输出所有元素
 * 后置条件：单链表不变
 */

void LinkSearch::PrintList( )
{
    Node *p;
    p=first->next;
    cout<<"单链表中的元素为:"<<endl;
    while (p)
    {
        cout<<p->data<<"  ";
        p=p->next;
    }
}

/*
 * 单链表的顺序查找
 */
int LinkSearch::SeqSearch2(Node *first, int k)
{
    Node *p;
    int count=0;
    p=first->next;
    int j=1;
    while ( p->data != k)
    {
        p=p->next;
        j++;count++;
    }
    if (p->data==k){
        
        cout<<"\n"<<"比较的次数为:"<<count<<endl;
        return j;
    }
    else{
        
        cout<<"比较的次数为:"<<count<<endl;
        return 0;
    }
}
/*
 * 取头节点
 */
Node* LinkSearch::GetFirst()
{
    return first;
}
```

### 2.顺序表的顺序查找
```c++
int SeqSearch1(int r[], int n, int k)
{
    r[0]=k ;          //设置哨兵
    int i=n,count=0;
    while (r[i]!=k)   //若r[i]与k相等，则返回当前i的值;否则继续比较前一个记录;
    {
        i--;
        count++;
    }
    cout<<"\n"<<"比较的次数为"<<count<<endl;
    return i;
}
```

### 3.折半查找递归
```c++
int BinSearch2(int r[], int low, int high, int k,int count)
{
    if (low>high){
        
        cout<<"比较的次数为:"<<++count<<endl;
        return 0;   //递归的边界条件
    }
    else
    {
        int mid=(low+high)/2;
        
        if (k<r[mid])
            
            return BinSearch2(r, low, mid-1, k,++count);     //查找在左半区进行
        
        if (k>r[mid])
            
            return BinSearch2(r, mid+1, high, k,++count);    //查找在右半区进行
        
        if (k==r[mid]){
        cout<<"比较的次数为:"<<++count<<endl;
            return mid;
        }   
    }
    return 0;
}
```