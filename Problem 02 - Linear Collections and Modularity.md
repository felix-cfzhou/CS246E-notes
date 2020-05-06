# Problem 2: Linear Collections + Modularity

- linked lists + arrays

### Linked List:

#### node.h

```cpp
#include <cstddef>


struct Node {
    int data;
    Node *next;
};

size_t size(Node *n); // how long is this linked list
// size_t is garanteed to hold any size in memory and is unsigned
```

#### node.cc

```cpp
#include <cstddef>
#include "node.h"


size_t size(Node *n) {
    size_t count=0;
    for(Node *curr=n; curr; curr=curr->next) ++count;
    return count;
}
```

#### main.cc

```cpp
#include "node.h"


int main() {
   	// do not use malloc/free in c++
    // use new/delete :)

    Node *n = new Node;
    n->data = 3;
    n->next = nullptr;
    // nullptr
    // garantees to be pointer compatible
    // NOT compatible with integers

    Node *n2 = new Node {3, nullptr};

    Node *n3 = new Node {4, new Node {5, nullptr}};
    
    delete n;
    delete n2;
    
    delete n3->next;
    delete n3;
    // OR
    while (n3) {
        Node *temp = n3;
        n3 = n3->next;
        delete tmp;
    }
}
```

## What happens when .../ When might ... happen?

```cpp
#include "node.h"
#include "node.h"
// might be included by including a header which already includes node.h


int main() {...}

// Doubly defined struct definition WILL NOT compile
// struct defined twice
```

### How Can we prevent it?

### C Preprocessor

- transforms program before the compiler sees it

ie

```cpp
#include ...
```

```cpp
#include <stdio.h>
#include <cstdio>

#define VAR VALUE
/*
preprocessor variable
all occurrences of VAR replaced with VALUE
*/

#define MAX 10 // Let's just not :(
int x[MAX]; // transformed to int x[10];
```

OR

#### myprogram.cc

```cpp
int main() {
    int x[MAX];
    ...
}
```

```bash
g++14 -DMAX=10 myprogram.cc
```

```cpp
#if SECURITYLEVEL == 1
	short int
#elif SECURITYLEVEL == 2 // take one of two to present to compiler
	long long int
#endif
	publickey;
```

**Special Case:**

```C++
#if 0  // industrial-strength "comment out"
...    // /* ... */ doesn't nest
#endif // Does nest 
```

Fixing the include problem: `#include guards`

_node.h_
```C++
#ifndef NODE_H  // If NODE\_H is not defined"
#define NODE_H  // Value is the empty string
... (file contents)
#endif
```

First time _node.h_ is included, symbol NODE_H not defined, so file is included.

Subsequently, symbol is defined, so file is supressed.

**ALWAYS**

- Put `#include guards` in header files

**NEVER**

- Compile.h files
- Include .cc files

Now what if someone writes:

```C++
struct Node {
   int data;
   Node *left, *right; 
};

size_t size(Node *n); // Size of tree
```

You can't use both in the same program

**Solution:** namespaces

_list.h_

```C++
namespace List {
    struct Node {
        int data;
        Node *next;
    };

    size_t size(Node *n); 
}
```

_tree.h_

```C++
namespace Tree {
    struct Node {
        int data;
        Node *left, *right;
    };

    size_t size(Node *n); 
}
```

_list.cc_

```C++
#include "list.h"

size_t List::size(Node *n) {  // List::Node *n is not necessary b/c the function is in the namespace
    ...
}
```

_tree.cc_

```C++
#include "tree.h"

size_t Tree::size(Node *n) { 
    ...
}
```

**Namespaces are open**

Anyone can add items to any namespace

_some\_other\_file.h_

```C++
namespace List {
    struct some_other_struct {...};
}
```