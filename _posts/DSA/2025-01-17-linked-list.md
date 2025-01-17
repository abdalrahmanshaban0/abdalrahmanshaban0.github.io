---
layout: post
title: Linked List
date: 2025-01-17 21:22 +0200
category: [DSA]
tags: [Data Structures And Algorithms, Linked List]
---
## Singly Linked List
A list of nodes in **Linear** form each node has data and a pointer to the next node (NULL if there's no nodes).

```cpp
template <typename T>
struct SNode
{
	T elem;
	SNode<T> *next;
};

template <typename E>
class SLinkedList
{
public:
	SLinkedList() : head(NULL) {}
	~SLinkedList()
	{
		while (!empty())
		{
			removeFront();
		}
	}
	bool empty() const
	{
		return head == NULL;
	}
	const E &front() const
	{
		return head->elem;
	}
	void addFront(const E &e)
	{
		SNode<E> *nd = new SNode<E>;
		nd->elem = e;
		nd->next = head;
		head = nd;
	}
	void removeFront()
	{
		SNode<E> *old = head;
		head = head->next;
		delete old;
	}

private:
	SNode<E> *head;
};
```
## Doubly Linked List
Each node has a pointer to its next and another to its previous. To ease the implementation we'll use dummy nodes as header and trailer.
```cpp
template <typename T>
struct DNode
{
	T elem;
	DNode<T> *next;
	DNode<T> *prev;
};

template <typename E>
class DLinkedList
{
public:
	DLinkedList()
	{
		header = new DNode<E>;
		trailer = new DNode<E>;
		header->next = trailer;
		trailer->prev = header;
	}
	~DLinkedList()
	{
		while (!empty())
		{
			removeFront();
		}
		delete header;
		delete trailer;
	}
	bool empty() const
	{
		return header->next == trailer;
	}
	const E &front() const
	{
		return header->next->elem;
	}
	const E &back() const
	{
		return trailer->prev->elem;
	}
	void addFront(const E &e)
	{
		add(header->next, e);
	}
	void addBack(const E &e)
	{
		add(trailer, e);
	}
	void removeFront()
	{
		remove(header->next);
	}
	void removeBack()
	{
		remove(trailer->prev);
	}

protected:
	void add(DNode<E> *v, const E &e)
	{ // insert new node before v
		DNode<E> *u = new DNode<E>;
		u->elem = e;
		u->next = v;
		u->prev = v->prev;
		v->prev->next = u;
		v->prev = u;
	}
	void remove(DNode<E> *v)
	{ // remove node v
		// u <-> v <-> w
		DNode<E> u = v->prev;
		DNode<E> w = v->next;
		u->next = w;
		w->prev = u;
		delete v;
	}

private:
	DNode<E> *header, *trailer;
};
```

## Circularly Linked List
Just a singly linked list but the tail points to the head instead of pointing to NULL.
The node referenced by the **cursor** is called the back, and the node immediately following is called the front.
```cpp
template <typename E>
class CircleList
{
public:
	CircleList() : cursor(NULL) {}
	~CircleList()
	{
		while (!empty())
		{
			remove();
		}
	}
	bool empty() const
	{
		return cursor == NULL;
	}
	const E &front() const
	{
		return cursor->next->elem;
	}
	const E &back() const
	{
		return cursor->elem;
	}
	void advance() // The front is pushed back
	{
		cursor = cursor->next;
	}
	void add(const E &e) // Insert after cursor
	{
		SNode<E> *v = new SNode<E>;
		v->elem = e;
		if (cursor == NULL)
		{
			cursor = v;
			cursor->next = cursor;
		}
		else
		{
			v->next = cursor->next;
			cursor->next = v;
		}
	}
	void remove()
	{ // remove the node after cursor
		SNode<E> *temp = cursor->next;
		if (temp = cursor)
		{ // only one node
			cursor = NULL;
		}
		else
		{
			cursor->next = cursor->next->next;
		}
		delete temp;
	}

private:
	SNode<E> *cursor;
};
```