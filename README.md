---
description: >-
  This page is about the reason for creating these notes, what you'll gain from
  these notes and more!
---

# What is Huginn Notes ?

Drawing inspiration from the epic stories of Norse mythology, Huginn and Muninn are a pair of ravens that traverse the world, Midgard, bringing information to the god Odin. We believe this perfectly encapsulates what pentesting is all about!

Huginn and Muninn symbolize the Enumerating, Information Gathering, and Footprinting phase of our pentesting journey, which we consider the most crucial step in conducting a successful pentest.

## Why Huginn Notes?

There are great notes, cheatsheets, and checklists done by professionals and experts for literally everything a pentester needs. But we found that they all lack the all-in-one format for a newcomer to this demanding field (similar to us). So, in the search for specific information, one might lose a LOT of time looking for the wrong information in the wrong place.

That's where we come into play!&#x20;

#### Our Goal:

After much struggle at the start of our journey, we finally found our equilibrium, and we aim to help you find yours. Here's how: We did not only include walkthroughs, write-ups, and notes we took through the CPTS Job Role Path, we provided much more!

We emphasize the way one should look for information on the internet and the way one should learn as much as possible from the HTB modules, not just go through them to get the badges; because, in our opinion, they aren't that beginner-friendly despite being full of high-quality information. So we'll do our best to fill the gaps and connect the dots a beginner will most certainly stumble upon.

We selected what we think is 'enough' for a beginner to understand the basics and methodologies used all the way to the end of the path, with a great effort to simplify complex concepts using illustrations to visualize them.

{% hint style="warning" %}
* Our notes should NEVER be taken for granted! If something works in a certain scenario, it doesn't mean it works everywhere and anytime.
* Always consult additional resources. It is never enough; and that's what's fascinating about the field.
{% endhint %}

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_STATE 10;

typedef struct Process {
	long PID = -1;
	long PPID = -1;
	int size;
	int burst_time = 0;
	int arrival_time = 0;
	int remaining_time = 0;
	char* state = "NEW";
	process *next = NULL;
	process *previous = NULL;
}process;

process *head = (process*)malloc(sizeof(Process));
process *tail = (process*)malloc(sizeof(Process));

void init_list(void) {
	head->next = tail;
	tail->previous = head;
}

void mem_fail(void) {
	fprintf(stderr, "MEMORY ALLOCATION FAILED\n");
	fprintf(stdout, "EXIT STATUS CODE: %d\n", errno);
}

void push(long pid) {
	process *node = (process*)malloc(sizeof(Process));
	if (!node) {
		mem_fail();
		exit(EXIT_FAILURE);
	}
	
	node->PID = pid;
	printf("\nBURST TIME: ");
	scanf("%d", &node->burst_time);
	print("\nSIZE: ");
	scanf("%d", &node->size);
	printf("\nParent Process ID: ");
	scanf("%d", &node->PPID);

	process *current = head;
	while(current->next != null){
		current = current->next;
	}
	current->next = node;
	node->previous = current;
	tail = node;
}

int find(long pid){
	int pos = -1;
	if (head->next == tail) return pos;
	
	process *current = head;
	while (current->next != NULL){
		pos ++;
		if (current->PID == pid) return pos;
		current = current->next;
	}
	if (tail->PID == pid) return ++pos;
	return -1; 
}

int num_processes(void){
	int count=0;
	if (head->next == tail) return count;
	process *current = head;
	while(current->next != NULL){
		count++;
		current = current->next;
	}
	return count;
}


void pop(long pid){

}


```

