---
layout: post
title: Iterators
date: 2025-01-19 12:39 +0200
category: [DSA]
tags: [Data Structures And Algorithms, Iterators]
---
Iterator is an object used to access and iterate elements of a container (data structure) like (vectors, lists, etc).

Let's see an example of implementing doubly linked list with iterators:
## List ADT with Iterators
![list iterator](/assets/posts/DSA/listIterator.png)

```cpp
template <typename E>
class NodeList
{
private:
  struct Node
  {
    E elem;
    Node *next;
    Node *prev;
  };

public:
  class Iterator
  {
  public:
    E &operator*()
    {
      return v->elem;
    }
    bool operator==(const Iterator &p) const
    {
      return v == p.v;
    }
    bool operator!=(const Iterator &p) const
    {
      return v != p.v;
    }
    NodeList::Iterator &operator++()
    {
      v = v->next;
      return *this;
    }
    NodeList::Iterator &operator--()
    {
      v = v->prev;
      return *this;
    }
    friend class NodeList;

  private:
    Node *v;
    Iterator(Node *u)
    { // private constructor to make Iterator instances
      // only be created using NodeList (as it's a friend class).
      v = u;
    }
  };

public:
  NodeList()
  {
    n = 0;
    header = new Node;
    trailer = new Node;
    header->next = trailer;
    trailer->prev = header;
  }
  ~NodeList()
  {
    while (!empty())
    {
      eraseFront();
    }
    delete header;
    delete trailer;
  }

  int size() const
  {
    return n;
  }
  bool empty() const
  {
    return n == 0;
  }
  NodeList::Iterator begin() const
  {
    return Iterator(header->next);
  }
  NodeList::Iterator end() const
  {
    return Iterator(trailer);
  }
  void insert(const NodeList::Iterator &p, const E &e) // insert e before p
  {
    Node *v = p.v;
    Node *u = new Node;
    u->elem = e;
    u->next = v;
    u->prev = v->prev;
    v->prev->next = u;
    v->prev = u;
    n++;
  }
  void insertFront(const E &e)
  {
    insert(begin(), e);
  }
  void insertBack(const E &e)
  {
    insert(end(), e);
  }
  void erase(const NodeList::Iterator &p) // remove p
  {
    Node *v = p.v;
    // u <-> v <-> w
    Node *u = v->prev;
    Node *w = v->next;
    u->next = w;
    w->prev = u;
    n--;
    delete v;
  }
  void eraseFront()
  {
    erase(begin());
  }
  void eraseBack()
  {
    erase(--end());
  }

private:
  int n; // number of elements
  Node *header, *trailer;
};
```

You can test this implementation with this [problem](https://leetcode.com/problems/palindrome-linked-list/) on **LeetCode**.

This is my solution to test it:
```cpp
class Solution {
    NodeList<int> lst;
public:
    bool isPalindrome(ListNode* head) {
        while(head){
            lst.insertBack(head->val);
            head = head->next;
        }
        NodeList<int>::Iterator l = lst.begin(), r = --lst.end();
        int cnt = lst.size()/2;
        while(cnt--){
            if(*l != *r) return false;
            ++l;
            --r;
        }
        return true;
    }
};
```

## Sequence ADT using array
A sequence is an ADT that generalizes the [vector](https://abdalrahmanshaban0.github.io/posts/vector-dynamic-array/) and list ADTs with support to access elements by their index or their position.

Suppose we want to implement a sequence **S**. We store each element **e** in **arr[i]** and define an **Iterator** that contains index **i** and a reference to **arr**.
A major drawback with this approach is that the cells in **arr** have no way to reference their corresponding iterators. For example, after performing an **insertFront** operation, the elements have been shifted to new positions, but we have no way of informing the existing iterators for **S** that the associated positions of their elements have changed.

So, the alternate solution is that we'll store in **arr[i]** a pointer to an **Iterator** and store the data inside the **Iterator**. So, **Iterator** now contains index and value.

![array-based implementation of the sequence ADT](/assets/posts/DSA/ArraySequence.png)

```cpp
class Sequence
{
  struct Position
  {
    int idx;
    E elem;
  };

public:
  class Iterator
  {
  public:
    E &operator*()
    {
      return p->elem;
    }
    bool operator==(const Iterator &it) const
    {
      return p->idx == it.p->idx;
    }
    bool operator!=(const Iterator &it) const
    {
      return p->idx != it.p->idx;
    }
    Sequence::Iterator &operator++()
    {
      p = A[p->idx + 1];
      return *this;
    }
    Sequence::Iterator &operator--()
    {
      p = A[p->idx - 1];
      return *this;
    }
    friend class Sequence;

  private:
    Position *p;
    Position **A;
    Iterator(Position **arr, int i)
    {
      A = arr;
      p = A[i];
    }
  };

public:
  Sequence()
  {
    capacity = n = 1;
    arr = new Position *[1];
    arr[0] = new Position;
    arr[0]->idx = 0;
  }
  ~Sequence()
  {
    for (int i = 0; i < n; i++)
    {
      delete arr[i];
    }
    delete[] arr;
  }
  int size() const
  {
    return n - 1;
  }
  bool empty() const
  {
    return size() == 0;
  }
  Sequence::Iterator begin() const
  {
    return Iterator(arr, 0);
  }
  Sequence::Iterator end() const
  {
    return Iterator(arr, size());
  }
  E &operator[](int i)
  {
    return arr[i]->elem;
  }
  E &at(int i)
  {
    if (i < 0 || i >= size())
    {
      throw runtime_error("Illegal index of function at()");
    }
    return arr[i]->elem;
  }
  void erase(int i)
  {
    delete arr[i];
    for (int j = i + 1; j < n; j++)
    {
      (arr[j]->idx)--;
      arr[j - 1] = arr[j];
    }
    n--;
  }
  void insert(int i, const E &e)
  {
    if (n >= capacity)
    {
      reserve(max(1, 2 * capacity));
    }
    for (int j = n - 1; j >= i; j--)
    {
      (arr[j]->idx)++;
      arr[j + 1] = arr[j];
    }
    arr[i] = new Position;
    arr[i]->idx = i;
    arr[i]->elem = e;
    n++;
  }
  void reserve(int N)
  {
    if (capacity >= N)
      return;
    Position **newArray = new Position *[N];
    for (int i = 0; i < n; i++)
    {
      newArray[i] = arr[i];
    }
    if (arr != NULL)
      delete[] arr;
    arr = newArray;
    capacity = N;
  }

private:
  Position **arr;
  int capacity; // current array size;
  int n;        // number of elements
};
```