# In-Progress...

{% hint style="info" %}
<mark style="color:blue;">This document will be published upon completion of the module</mark>
{% endhint %}

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

#define MAX_NAME 21
typedef struct student{
	long id;
	float average;
	char name[MAX_NAME];
	student *next=NULL;
}student;

student *head=NULL;

void init(void){
	printf("1) Show students' list.\n");
	printf("2) Add student.\n");
	printf("3) Remove student.\n");
	printf("4) Sorst list..\n");
	printf("5) Search student.\n");
	printf("6) Save to a single file.\n");
	printf("7) Save in categories\n");
}

void mem_fail(void){
	fprintf(stderr,"Can't allocate memory...\n");
	fprintf(stdout,"Error code = %d\n", errno);
}

void add_head(long id){
	float avg;
	char name[MAX_NAME];
	printf("\nAVG:");
	scanf("%f", &avg);
	printf("\nNAME:");
	fgets(name, MAX_NAME, stdin);

	if (!head){
		head->average=avg;
		head->name=name;
		head->id=id;
		return;
	}
	
	student *node = (student*)malloc(sizeof(student));
	if (!node){
		mem_fail();
		exit(EXIT_FAILURE);
	}

	node->name=name;
	node->average=avg;
	node->id=id;
	node->next=head;
	head=node;
}

void add_tail(long id){
	float avg;
	char name[MAX_NAME];
	printf("NAME: ");
	fgets(name, MAX_NAME, stdin);
	printf("AVERAGE");
	scanf("%f", &avg);
	
	student *node = (student*)malloc(sizeof(student));
	if(!node){
		mem_fail();
		exit(EXIT_FAILURE);
	}
	node->average = avg;
	node->name = name;
	node->id = id;
	
	student *current;
	if (!current){
		mem_fail();
		exit(EXIT_FAILURE);
	}
	current = head;
	while(current->next != NULL){
		current = current->next;
	}
	current->next = node;
}

int find_student(long cin){
	student *current;
	if (!current){
		mem_fail();
		exit(EXIT_FAILURE);
	}
	current=head;

	while (current->next != NULL){
		if (current->id == id){
			return 1;
		}
	}
}


int main(void){

}
```

