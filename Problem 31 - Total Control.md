# roblProblem 31 - Total Control

- C++ allows you control in pretty much everything is done
  - copies
  - parameter passing
  - initialization
  - method call resolution
  - etc
- How bout allocation of memory?
  - Why?
  - How?
- Why do you need a custom allocator?
  - built-in one is too slow!!
    - the one we get with compiler is GENERAL PURPOSE
      - not optimized for any specific use
        - always allocate objects of the same size?
        - better localization?
        - special memory for hardware?
        - etc

 Recall

## Allocater

- built in one is too slow
- optimize locality
- to use special memory
- to profile program (collect stats)

# November 28, 1018

## How to customize allocation?

- overload `operator new`
- if you write a global operator new, all allocations in the program will use your allocator
- also write`operator delete`
  - else Undefined Behavior

```cpp
void *operator new(size_t size) {
    cout << "Request for " << size << "bytes \n";
    return malloc(size);
}

void operator delete(void *p) {
    cout << "Freding" << p << endl;
    free(p);
}

int main() {
    int *x = new int;
    delete x;
}
```

- works but is not correct
  - does not adhere to convention
- If `operator new` fails it is supposed to throw `bad_alloc`
- actually
  - if `operator new` fails, it is supposed to call the `new_handler` function
- `new_handler` function
  - can free up some space
    - (somehow)
  - install a different `new_handler` or de-install the current
  - throw `bad_alloc`
  - abort / exit
- `new_handler` should be called in an infinite loop
- if new handler is nullptr, then operator new throws
- Also
  - new must return a valid ptr if `size==0` and
  - `delete nullptr` must be safe

```cpp
#include <new>

void *operator new(size_t size) {
    cout << "Request for " << size << "bytes \n";
    while(true) {
        void *p = malloc(size);
        if(p) return p;
        std::new_handler h = std::get_new_handler();
        if(h) h();
        else throw std::bad_alloc {};
    }
}

void operator delete(void *p) {
    if(!p) return;
    cout << "Freeing " << p << endl;
    free(p);
}
```

- Replacing global operator new / delete affects entire program
- probably want to replace these operators on a class_by_class basis
  - especially if optimizing for the sizes of your objects
- define operator new and delete within the class
  - must be static members

```cpp
class C {
    public:
    static void *operator new(size_t size) {
        cout << "Running C's allocator \n";
        return ::operator new(size);
    }

    static void operator delete(void *p) noexcept {
        cout << "Freeing " << p << endl;
        return ::operator delete(p);
    }
};
```

- can we write to an arbitrary stream?

```cpp
class C {
    public:
    static void *operator new(size_t size, std::ostream &out) {
        out << ... << endl;
        return ::operator new(size);
    }
    ...;
};

C *x = new(cout) c;
ifstream f{...};
C *y = new(f) c;
```

- we need the corresponding delete

```cpp
class C {
    ...;
    static void operator delete(void *p, std::ostream &out) noexcept {
        out << ... << endl;
        ::operator delete(P);
    }
};

// WILL NOT COMPILE, need ordinary delete
```

```cpp
class C {
    ...;
    static void operator delete(void *p) noexcept {
        // as before
    }
};

C *p = new(f) C; // ofstream allocator
delete p; // ordinary delete
```

- must have ordinary delete, else compile error
- how can we call the specialized one?
  - cannot!
- why do we need it?
- If the ctor that runs after specialized new
  - throws
    - specialized operator new is called
- If there is not one
  - no delete is called
  - leak