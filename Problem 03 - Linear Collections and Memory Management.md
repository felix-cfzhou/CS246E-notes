# Problem 3: Linear Collections and Memory Management
**2017-09-14**  
**Readings:** 7.7.1, 14, 16.2 

**Arrays**
`int a[10];`

- On the stack, fixed size

On the heap:
`int *p = new int[10];`

To delete:
`delete[] p;`

Use `new` with `delete`, and `new [...]` with `delete[]`

Mismatching these is undefined behaviour

**Problem:** what if our array isn't big enough
Note: no `realloc` for `new`/`delete`  

Use abstraction to solve the problem:

_vector.h_

```C++
#ifndef VECTOR_H
#define VECTOR_H

namespace CS246E {
    struct vector {
        size_t size, cap;
        int *theVector;
    }
};

const size_t startsize = 1;

vector make_vector();

size_t size(const vector &v);

int &itemAt(const vector &v, size_t i);

void push_back(const vector &v, int x);

void pop_back(const vector&v);

void dispose(vector &v);

#endif
```

_vector.cc_

```C++
#include "vector.h"

namespace {  // Anonymous namespace makes the function only visible to file (same as static in C)
    void increaseCap(CS246E::vector &v) {
        if (v.size == v.cap) {
            int *newVec = new int[2 * v.cap];

            for (size_t i = 0; i < v.cap; ++i) {
                newVec[i] = v.theVector[i]
            }

            delete[] v.theVector;
            v.theVector = newVec;
            v.cap *= 2;
        }
    }
}

CS246E::vector CS246E::make_vector() {
    vector v {0, startSize, new int[startSize]};
    return v;
}

size_t CS246E::size(const vector &v) {
    return v.size;
}

int &CS246E::itemAt(const vector &v, size_t i) {
    return v.theVector[i];
}

void CS246E::push_back(vector &v, int n) {
    increaseCap(v);
    v.theVector[v.size++] = n;
}

void CS246E::pop_back(vector &v) {
    if (v.size > 0) {
        --v.size;
    }
}

void CS246E::dispose(vector &v) {
    delete[] v.theVector;
}
```

# Recall

```cpp
#ifndef VECTOR_H
#define VECTOR_H
namespace CS246E{
    struct Vector {
        int *theVector;
        size_t size, cap;
    };
    
    const size_t startSize = 1;
    Vector makeVector();
    size_t size(const vecctor &v);
    int &itemAt(const Vector &v, size_t i);
    void push_back(Vector &v, int n);
    void pop_back(Vector &v);
    void dispose(Vector &v);
}
#endif
```

# September 19, 2018

## main.cc

```cpp
#include "vector.h"


using CS246E::Vector;

int main() {
    Vector v = CS246e::makeVector();
    push_back(v, 1);
    push_back(v, 10);
    push_back(v, 100);
    // no need to indicate CS246E::push_back, CS246E::itemAt, CS246E::dispose?
    
    itemAt(v, 0);
    dispose(v);
}
```

## Argument-Dependent Lookup (ADL) / Koenig Lookup

- if type of argument belongs in specific namespace, we can look in specific namespace
- if the type of a function f's argument belongs  to a namespace n, then C++ will search namespace n, as well as the current scope for a function matching f
- this is why << to std::cout is not prefixed by std namespace

```cpp
std::cout << x
std::operator<<(std::cout, x);
```

## Problems with out Vector

- what if client does not use makeVector?
  - uninitialized object
- what if we go not call dispose?
  - memory leak?
- How can we make this more robust?

## Intro to Classes

- fundamentally, a class guarantees proper initialization and disposal

### Function Inside Structs

Class is just a structure which contain functions?

```cpp
struct Student {
    int assns, mt, final;
    float grade() {
        return assns*0.4 + mt*0.2 + final*0.4;
    }
}
```

- functions inside structs called member function or methods
- instances of a class called objects

```cpp
Student bob { 90, 70, 80 };
// Student is class
// bob is object

cout << bob.grade(); // methods
```

- What do assns, mt, final mean within grade() {...}

- fields of the current object

  - the receiver of the method call, ie bob

  #### Method vs Function

  - methods differ from functions in that methods take an implicit parameter called this, a ptr to the receiver object

```cpp
bob.grade();
// =>
// this == &bob

Struct Student {
    float grade() {
        return this->assns*0.4 + this->mt*0.2 + this->final*0.4;
    }
}
```

### Initializing Objects

```cpp
Student bob { 90, 70, 80 }; // C-style struct initialization, OK -> but limited
```

### Constructors

```cpp
struct Student {
    int assns, mt, final;
    
    Student(int assns, int mt, int final) {
        this->assns = assns;
        this->mt = mt;
        this->final = mt;
    }
};

// Now
Student bob { 90, 70, 80 };
// calls the constructor with args 90, 70, 80

Student *p = new Student { 90, 70, 80 };
```

#### Note:

- once a constructor is defined, C-style default field by field initialization is automatically removed
- what do assns, mt, final mean within grade?
  - fields of cur

# Recall

## vector.h

```cpp
#ifndef VECTOR_H
#define VECTOR_H
namespace CS246E {
    struct Vector {
        size_t size, cap;
        int *theVector;
        
        Vector();
        size_t size();
        int &itemAt(size_t i);
        void push_back(int n);
        void pop_back();
        ~Vector();
    };
}
#endif
```

## vector.cc

```cpp
#include "vector.h"

namespace {
    void increaseCap(Vector &v) {...}
    const size_t startSize = 1;
}

CS246E::Vector::Vector(): size{0}, cap{startSize}, theVector{new int [cap]} {}
size_t CS246E::Vector::size() {return n;}
CS246E::Vector::~Vector() {delete[] theVector;}
```

## main.cc

```cpp
int main() {
    Vector v; // constructor automatically called
    v.push_back(1);
    v.push_back(10);
    v.push_back(100);
    v.itemAt(0) = 2;
} // no dispose - dtor cleans v up
```