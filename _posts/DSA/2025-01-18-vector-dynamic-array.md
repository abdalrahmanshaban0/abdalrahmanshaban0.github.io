---
layout: post
title: Vector (Dynamic Array)
date: 2025-01-18 23:57 +0200
category: [DSA]
tags: [Data Structures And Algorithms, Vector]
---
The simple idea of dynamic array is the ability to resize itself when inserting new elements to avoid overflow.

How we can extend the array (make it grows)?
- Create an initial size **capacity** for the array.
- When number of elements **n** exceeds **capacity**, make a new array of size **2*capacity**, move old array elements to the new one and deallocate the old array's memory.
![Resizing Array](/assets/posts/DSA/ArrayVector.png)

```cpp
template <typename E>
class ArrayVector
{
public:
  ArrayVector()
  {
    capacity = n = 0;
    arr = NULL;
  }
  int size() const
  {
    return n;
  }
  bool empty() const
  {
    return size() == 0;
  }
  E &operator[](int idx)
  {
    return arr[idx];
  }
  E &at(int idx)
  { // just like [] but with boundary protection
    if (idx < 0 || idx >= n)
    {
      throw runtime_error("Illegal index in function at()");
    }
    return arr[idx];
  }
  void erase(int idx)
  {
    for (int i = idx + 1; i < n; i++)
    {
      arr[i - 1] = arr[i];
    }
    n--;
  }
  void insert(int idx, const E &e) // inserted element will be at idx
  {
    if (n >= capacity)
    {
      reserve(max(1, 2 * capacity)); // 1 because capacity will be initially 0
    }
    for (int i = n - 1; i >= idx; i--)
    {
      arr[i + 1] = arr[i];
    }
    arr[idx] = e;
    n++;
  }
  void reserve(int N)
  {
    if(capacity >= N) return;
    E* newArray = new E[N];
    for(int i = 0; i < n; i++){
      newArray[i] = arr[i];
    }
    if(arr != NULL) delete [] arr;
    arr = newArray;
    capacity = N;
  }

private:
  E *arr;
  int capacity; // current array size
  int n;        // number of elements
};
```