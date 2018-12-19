# Problem 14 - Memory Management is Hard!

No it is not

## Vectors

- do everything arrays can do
- grow as needed
  - O(1) amortized time or better
- auto clean when out of scope
- tuned to minimize copying

### Just use vectors

- never have to manage arrays again

## The Point

- c++ has enough abstraction facilities to make programing _easier_ than c

## But what about single objects?

```cpp
void f() {
    Posn *p = new Posn {1, 2};
    ...;
    delete p; // must deallocate the Posn!
}
```

### First ask: Do we really need to use the heap?

- use stack instead?

```cpp
void f() {
    Posn p {1, 2};
    ...;
} // no clean up necessary
```

### Sometimes you do need the heap

- calling delete is not so bad

```cpp
class BadNews {};
void f() {
    Posn *p = new Posn{1, 2};
    if(some_condition) {throw BadNews{};}
    delete P;
}
```

- note that p is leaked if f throws. Unacceptable. >:(

## Raising and Handling Exceptions Should NOT corrupt the program

- we desire __exception safety__

### Leaks are corruption of the program heap

- eventually degrade performance and crash the program
- If a program cannot recover from and exception without corrupting its memory, what is the point of recovering?

## What constitutes exception safety

1. Basic guarantee
   1. once an exception is handled, the program is in a valid state
   2. No leaked memory, no corrupted data structures, all invariants maintained
2. Strong guarantee
   1. If an exception propagates outside a function f, the the state of the program is as if f had not been called
3. No throw guarantee
   1. a function f offers no throw guarantee if it __NEVER__ emits an exception and __always__ accomplishes its purpose
4. Will Revisit

## Coming Back to....

```cpp
void f() {
    Posn *p = new Posn {1 2};
    if(some_condition) {
        delete p;
        throw BadNews {};
    }
    
    delete p;
}
```

- delete twice, duplicated effect, more lines
  - memory management is even harder than before!

### We Wish to delete p no matter what

- what guarantees does c++ offer?
  - dtors for stack allocated objects will be called when the objects go out of scope
    - IF exceptions are handled

## Create a class with a dtor that deletes the ptr

```cpp
template<typename T> class unique_ptr {
    T *p;
    public:
    unique_ptr(T *p): p{p} {}
    ~unique_ptr() {delete p;}
    T *get() const {return p;}
    T *release() {
        T *q = p;
        p = nullptr;
        return q;
    }
}
```

Now consider

```cpp
void f() {
    unique_ptr<Posn> p {new Posn {1, 2}};
    if(some_condition) {throw BadNews {};}
}
```

That is it!

- __less__ memory management effort than we started with :)

## unique_ptr

- can use get to fetch the ptr

### Better

- make unique_ptr act like ptr

```cpp
template<typename T> class unique_ptr {
    T *p;
    public:
    ...;
    T &operator*() const {
        return *p;
    }
    
    T *operator->() const { // returns ptr and built-in operator-> is called on that ptr
        return p;
    }
    
    explicit operator bool() {return p;} // explicit prohibits bool b = p
    
    void reset(T *p1) {
        delete p;
        p = p1;
    }
    
    void swap(unique_ptr<T> &x) {
        std::swap(p, x.p);
    }
}
```

```cpp
void f() {
    unique_ptr<Posn> p {new Posn {1, 2};}
    cout << p->x << " " << p->y;
}
```

### But consider

```cpp
unique_ptr<Posn> p {new Posn{1, 2};}
unique_ptr<Posn> q = p;
```

- two pointers to the same object
  - cannot both delete
    - probs results in crash

#### Solution

- copying unique_ptrs is __NOT ALLOWED__
- but can __MOVE__

```cpp
template<typename T> class unique_ptr {
    ...;
    public:
    ...;
    unique_ptr(const unique_ptr<T> &other) = delete;
    unique_ptr &operator=(const unique_ptr &other) = delete;
    unique_ptr(unqiue_ptr &&other): p{other.p} {other.p = nullptr;}
    unique_ptr &operator=(unique_ptr &&other) {
        swap(other);
        return *this;
    }
}
```

# Recall:

- unique_ptr
- copies disabled
- moves allowed

# Note

- small safety issue

```cpp
class C {...};
void f(unique_ptr<c> x, int y) {...}
int g() {...}
f(unique_ptr<C>{new C}, g());
```

- c++ does not enforce order of argument evaluation

  - may go beyond that

  - does NOT guarantee full evaluation before next parameter evaluated

1. new C
2. g()
3. unique_ptr\<c\> {(1)}

- what if g throws??
  - new C leaks

### Fix

- make 1 and 3 inseperable
  - put both into a helper function

```cpp
template<typename T, typename... Args> unique_ptr<T> make_unique(Args&&... args) {
    return unique_ptr<T> {new T(std::forward<Args>(args)...)};
}

// Now
f(make_unique<T>(), g());
```

- no leak if g throws :)

## Resource Acquisition Is Initialization (RAII)

unique_ptr is an example of the C++ idiom named above

- any resource that must be properly released should be wrapped in a stack allocated object whos dtor frees it
  - memory
    - unique_ptr
  - file
    - ifstream
  - handle
  - etc
- acquiring a resource as part of the object initialization so it can be released when the object dtor runs