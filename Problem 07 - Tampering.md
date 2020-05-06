# Problem 7: Tampering

```cpp
vector v;
v.cap = 100; // sets cap w/o allocating memory
v.push_back(1);
v.push_back(2); // does not have memory allocated for 100, undefined behavior
```

## Interfering with ADTS:

- forgery
  - creating an object without calling a ctor function
  - not possible once we wrote ctors
- tampering
  - accessing the internals without using a provided interface function

## Invariants

- statement that will always be true about an abstraction
- ADTs provide and rely on invariants

### ie

- stacks
  - LIFO
- vectors
  - rely on an invariant that 0 ~ cap-1 are valid to access

We cannot rely on invariants if the user can interfere

-  makes the program hard to reason about

# FIX: encapsulation

- sealing objects into "black boxes"

```cpp
namespace CS246E {
    struct Vector {
        private: // only visible within the vector itself
        size_t n, cap;
        int *theVector;
        
        public: // visible to all, default is no specifier given
        Vector();
        size_t size() const;
        void push_back(int n);
        ...;
    };
}
```

## vector.cc

```cpp
#include "vector.h"

namespace {
    void increaseCap(Vector &v) { ...; } // no longer works, no access to private internals
}
```

## vector.h

```cpp
namespace CS246E {
    struct Vector {
        private:
        size_t n, cap;
        int theVector[];
        
        public:
        Vector();
        ...;
        
        private:
        void increaseCap(); // Now a private method
        // no need for anonymous namespace anymore
    };
}
```

### Structs

- public default access
- private default access would be better

### Class

- exactly like struct BUT private default access

```cpp
class Vector {
    size_t n, cap;
    int *theVector;
    
    public:
    Vector();
    ...;
    
    private:
    void increaseCap();
};

// vector.cc same
```



Similar Problem

```cpp
Node n {3, nullptr}; // stack-allocated
Node n {4, &n}; // dtor for n will try to delete &n
```

There was an invariant:

- next is either nullptr or heap allocated

How can we enforce this?

- encapsulate entire Node class inside a wrapper class

```cpp
class List {
    struct Node {
        int data;
        Node *next;
        ...;
    }
    
    Node *theList;
    size_t n;
    
    public:
    list(): theList{nullptr} n{0} {}
    ~list() {delete theList;}
    size_t size() const;
    
    void push_front(int x) {
        theList = new Node {x, theList};
        ++n;
    }
    
    void pop_front() {
        if(theList) {
            Node *tmp = theList;
            theList = theList->next;
            
            tmp->next = nullptr;
            delete tmp;
            
            --n;
        }
    }
    
    const int &operator[](size_t i) const {
        Node *cur = theList;
        for(size_t j=0; j<i; ++j, cur=cur->next);
        return cur->data;
    }
    
    int &operator[](size_t i) {...;}
};
```

#### Note:

- since client cannot manipulate the list directly
  - no access to next ptrs
  - invariant is maintained