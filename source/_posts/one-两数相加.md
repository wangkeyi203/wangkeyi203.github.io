---
title: 两数相加
date: 2019-03-20 21:11:52
tags: 
- Algorithm
- 第一周
---

### 题目

两个给出  **非空**的链表用来表示两个非负的整数。其中，各自它们的位数的英文按照  **逆序**  的方式存储的，它们并且每个的节点只能存储  **一位**  数字。

如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。

您可以假设除了数字0之外，这两个数都不会以0开头。

**示例：**

```
输入：（2  - > 4  - > 3）+（5  - > 6  - > 4）
 输出： 7  - > 0  - > 8
 原因： 342 + 465 = 807
```

### 思路

循环判断，创建一个标志位和一个新表头，当两个链表都不为NULL时，新建一个node，初始化，新表头的next指向新node，连个链表的数值和标志位相加，存到当前node，并判断结果数值是否大于10，大于10则标志位置1，结果减10，为了结尾不出现0，判断两个链表的next是否为NULL，标志位是否为0，都满足则当前node的next置为NULL。

若有一个链表已经结束，则另一条链表数值和标志位依次相加，并重复上面判断数值部分，并在加上防止结尾出现0的操作。

### 代码

```c
struct ListNode* addTwoNumbers(struct ListNode* l1, struct ListNode* l2) {
    int a = 0;
    struct ListNode* n = (struct ListNode*)malloc(sizeof(struct ListNode));
    struct ListNode *p = l1;
    struct ListNode *q = l2;
    struct ListNode *e = n; //for return
    e->next = NULL; //init
    e->val = 0;
    while( p !=NULL && q != NULL)  
    {
        struct ListNode* newnode = (struct ListNode*)malloc(sizeof(struct ListNode));
        newnode->val =0;
        newnode->next=NULL;//init newnode
        e->next=newnode;
        e->val = q->val + p->val + a ;
        if(e->val >= 10)
        {
            e->val -= 10;
            a=1;
        }
        else a = 0;
        p=p->next;
        q=q->next;
        if(NULL==q && NULL==p && a==0) 
            e->next=NULL;
        e=e->next;    
    }
    while(q !=NULL)
    {
        struct ListNode* newnode = (struct ListNode*)malloc(sizeof(struct ListNode));
        newnode->val =0;
        newnode->next=NULL;//init newnode
        e->next=newnode;
        e->val = q->val + a;
       
        if(e->val >= 10)
        {
            e->val -= 10;
            a=1;
        }
        else a=0;
        q=q->next;
        if(NULL==q && a==0) 
            e->next=NULL;
        e=e->next;
        
    }
    while(p !=NULL)
    {
        struct ListNode* newnode = (struct ListNode*)malloc(sizeof(struct ListNode));
        newnode->val =0;
        newnode->next=NULL;//init newnode
        e->next=newnode;
        e->val = p->val + a;
        if(e->val >= 10)
        {
            e->val -= 10;
            a =1;
        }
        else a=0;
        p=p->next;
        if(NULL==p && a==0) 
            e->next=NULL;
        e=e->next;
    } 
    if (a!=0)
        e->val+=a;
    return n;
}
```
