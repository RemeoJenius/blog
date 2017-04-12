---
title: C语言链表
date: 2017-03-24 18:06:43
tags:
---
## 链表
## 代码 linked-list.c
``` c
#include "node.h"
#include <stdio.h>
#include <stdlib.h>

// typedef struct _node{
// 	int value;
// 	struct _node *next;
// }Node;

typedef struct _list{ // 没有做尾指针
	Node *head;
	Node *tail;
} List;


void add(List *list, int number);
void add2(List *list, int number);

int main(int argc, char const *argv[])
{
	List list;
	list.head = list.tail = NULL;
	int number;
	do{
		scanf("%d",&number);
		if(number != -1){
			add2(&list,number);
		}
	}while(number != -1);
	for (Node *q = list.head;q;q=q->next){
		printf("%d",q->value);
	}
	return 0;
}

void add(List *list, int number){
	// add to linked-list
	Node *p = (Node*)malloc(sizeof(Node));
	p->value = number;
	p->next = NULL;
	// find last node
	Node *last = list->head;
	if(last){
		while(last->next){
			last = last->next;
		}
		last->next = p;
	}else{
		list->head = p;
	}


}

void add2(List *list, int number){  // 哎，早画图早解决了服了
	// add to linked-list
	Node *p = (Node*)malloc(sizeof(Node));
	p->value = number;
	p->next = NULL;
	// find last node
	if(list->tail){
		(list->tail)->next = p;
		list->tail = (list->tail)->next;

	}else{
		list->head = list->tail = p;

	}

}
```
这是实现了基本的创建链表功能，有两个版本第一个版本(函数add())与第二个版本(add2())相比，第二个版本是具有为指针的每一次不用从头遍历到尾节点。
##删除加寻找
``` c
#include "node.h"
#include <stdio.h>
#include <stdlib.h>

// typedef struct _node{
// 	int value;
// 	struct _node *next;
// }Node;

typedef struct _list{ // 没有为检点的版本
	Node *head;
	Node *tail;
} List;

// typedef struct _list{
// 	Node *head;
// 	Node *tail;
// } List;



void add(List *list, int number);
void add2(List *list, int number);
void print(List *list);
void queryAndDelete(List *list, int number);

int main(int argc, char const *argv[])
{
	List list;
	list.head = list.tail = NULL;
	int number;
	do{
		scanf("%d",&number);
		if(number != -1){
			add2(&list,number);
		}
	}while(number != -1);
	print(&list);
	scanf("%d",&number);
	queryAndDelete(&list,number);
	print(&list);
	return 0;
}

void add(List *list, int number){
	// add to linked-list
	Node *p = (Node*)malloc(sizeof(Node));
	p->value = number;
	p->next = NULL;
	// find last node
	Node *last = list->head;
	if(last){
		while(last->next){
			last = last->next;
		}
		last->next = p;
	}else{
		list->head = p;
	}


}

void add2(List *list, int number){  // 哎，早画图早解决了服了
	// add to linked-list
	Node *p = (Node*)malloc(sizeof(Node));
	p->value = number;
	p->next = NULL;
	// find last node
	if(list->tail){
		(list->tail)->next = p;
		list->tail = (list->tail)->next;

	}else{
		list->head = list->tail = p;

	}

}

void print(List *list){
	for (Node *q = list->head;q;q=q->next){
		printf("%d\t",q->value);
	}
	printf("\n");
}
void queryAndDelete(List *list,int number){
	int isFound = 0;
	Node *p = NULL;
	for ( Node *q = list->head;q;p = q,q=q->next){
		if(q->value == number){
			isFound = 1;
			if(q){
				p->next = q->next;
			}else{
				(list->head)->next = q->next;
			}
			free(q);
			break;
		}
	}
	if(isFound){
		printf("找到了\n");
	}
}

```
