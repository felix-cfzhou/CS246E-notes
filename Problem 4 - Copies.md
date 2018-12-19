# Problem 4: Copies

# September 25, 2018

```cpp
Vector v;
v.push_back(100);
...;
Vector w = v; // shallow copy, uses SAME pointer to same array
w.itemAt(0); / 100
w.itemAt(0) = 200;
v.itemAt(0); // 200

// dtor of w frees theArray, v tries to free theArray too, gg
```

## Shallow Copy

- basic field by field copying
- v, w share the same data

```cpp
Vector w = v; // constructs w as a copy of v
```

- invokes the copy constructor

```cpp
struct Vector {
    ...;
    Vector(const Vector &other); // copy constructor
};
```

- compiler supplied copy constructor simply copies all fields
- deep copy
  - write your own copy ctor

```cpp
struct Node { // Vector class as excercise
    int data;
    Node *next;
    ...;
    Node(const Node &other):
    	data{other.data}, next{other.next ? new Node {*other.next} : nullptr} {}
};
```

```cpp
Vector v;
Vector w;
w = v;
/*
copy operation but NOT copy ctor, shallow
copy assignment operator
compiler supplied - copies each field (shallow)
*/
```

## Deep Copy Assignment

```cpp
struct Node {
    int data;
    Node *next;
    ...;
    Node &operator=(const Node &other) { // if call by value, recursive call to copy ctor
        data = other.data;
        delete next;
        next = other.next ? new Node {*other.next} : nullptr;
        
        return *this;
    }
};
// WRONG AND DANGEROUS

// Consider
Node n {...};
n = n; // destroys any data, then tries to copy :(
*p = *q;
a[i] = a[j];

Node &Node::operator=(const Node &other) { // Note that we return reference
    if(this == &other) return *this;
    // as before
}
```

## Alternative: copy + swap idiom

```cpp
#include <utility>
struct Node {
    ...;
    void swap(Node &other) {
        using std::swap;
        swap(data, other.data);
        swap(next, other.next);
    }
    
    Node &operator=(const Node &other) { // less efficient, always copies even if not necessary
        Node tmp = other; // requires working deep copy ctor and dtor
        swap(tmp); // not std::swap, own function
        
        // tmp goes out of scope and dtor called
        return *this;
    }
}
```