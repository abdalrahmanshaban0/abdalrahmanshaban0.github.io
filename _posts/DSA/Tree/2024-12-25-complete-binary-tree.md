---
title: Complete Binary Tree
classes: wide
header:
  teaser: /assets/images/DSA/complete-tree.png
ribbon: Green
description: "Complete binary tree is a binary tree where all of its level are filled completely except the last level is filled from left to right.
A binary tree is **perfect** if all parent nodes have two children and all leaves are on the same level.
The **level** of a node is the number of edges along the path between it and the root node."
categories: 
  - DSA
toc: true
---

Complete binary tree is a binary tree where all of its level are filled completely except the last level is filled from left to right.
A binary tree is **perfect** if all parent nodes have two children and all leaves are on the same level.
The **level** of a node is the number of edges along the path between it and the root node.
```cpp
template<typename E>  
class completeTree {  
public:  
    using Position = typename vector<E>::iterator;  
    int size() const {return v.size() - 1;}  
    Position left(const Position& p) const {return pos(2*idx(p));}  
    Position right(const Position& p) const {return pos(2*idx(p));}  
    Position Parent(const Position& p) const {return pos(idx(p)/2);}  
    bool hasLeft(const Position& p) const {return 2*idx(p) <= size();}  
    bool hasRight(const Position& p) const {return 2*idx(p)+1 <= size();}  
    bool isRoot(const Position& p) const {return idx(p) == 1;}  
    Position root() {return pos(1);}  
    Position last() {return pos(size());}  
    void addLast(const E& e) {v.push_back(e);}  
    void removeLast() {v.pop_back();}  
    void swap(const Position& p, const Position& q){swap(*p, *q);}  
private:  
    vector<E> v;
    int idx(const Position& p) {  
        return (p-v.begin());  
    }  
    Position pos(const int& i) {  
        return v.begin() + i;  
    }  
};
```
