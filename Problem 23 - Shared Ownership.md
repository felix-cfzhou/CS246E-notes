# Problem 23: Shared Ownership

Is C++ hard? NO (if you're a client programmer)

## Explicit Memory Management

- vector
- unique_ptr
- stack-allocated objects

following these, never say `delete`, `delete[]`

# Recall

- is c++ hard?
  - Not if you are client
- but `unique_ptr` does not respect "is-a"

# November 20, 2018

# Problem 23 - Shared Ownership

## This works

```cpp
unique_ptr<Base> p = unique_ptr<Derived> {new Derived {...}}; // OK
p->virt-fn(); // runs derived version ojbk :)
```

## BUT!

```cpp
unique_ptr<Derived> q = ...;
unique_ptr<Base> p = std::move(q); // ERROR!
```

- not conversion between `unique_ptr<Derived>` and `unqiue_ptr<Base>`
  - relationship does not carry over to template arguments

### Easy Fix

```cpp
template<typename T> class unique_ptr {
    ...;
    T *p;
    
    public:
    ...;
    template<typename U> unique_ptr(unique_ptr<U> &&q): p{q.p} {q.p = nullptr;}
    template<typename U> unique_ptr &operator=(unique_ptr<U> &&q) {
        std:swap(p, q.p);
        return *this;
    }
}
```

- works for any `unique_ptr` whose pointer is assignment compatible with `this->p`
  - eg
    - subtypes of `T` but NOT supertypes of `T`

## I want _TWO_ smart ptrs pointing at the same object!

### Recall - Racket

```scheme
(define l1 (cons 1 (cons 2 (cons 3 empty))))
(define l2 (const 4 (rest l1)))
```

- shared data structures are a nightmare in `C`
  - how can we ensure each node is freed?
  - easy in GC languages
- C++ answer?

## `shared_ptr`

```cpp
template<typename T> class shared_ptr {
    T *p;
    int *refcount;
    
    public:
    ...;
}
```

- `refcount`
  - counts how many `shared_ptrs` point to `*p`
  - updated each time a `refcount` is initialized, assigned, or destroyed
  - shared among all `share_ptr`s pointing to the same thing
  - `p` is only deleted when its `refcount` reaches 0
  - implementation details as exercise

### Shared List

```cpp
struct Node {
    int data;
    shared_ptr<Node> next; // voila
};
```

- deallocation is now easy

### HOWEVER!

#### Cycles

- having cyclic data sometimes requires us to physically break the cycle
  - or `weak_ptr`
    - like a `shared_ptr` except if `refcount` only 1, it is gone

#### Watch

```cpp
Book *p = new ...;
share_ptr<Book> p1 {p};
share_ptr<Book> p2 {p};
```

- 2 _separate_ `refcount`s so deletion happens twice :(
- `share_ptr`s are not mind readers
- `p1` and `p2` will not share refcount
  - __BAD!__
- if we want to point 2 `shared_ptr`s at an object:
  - create one `shared_ptr` and copy it

## But - cannot `dynamic_cast` `shared_ptr`s

- write our own!

```cpp
template<typename T, typename U> shared_ptr<T> dynamic_pointer_cast(const shared_ptr<U> &spu) {
    return shared_ptr<T>(dynamic_cast<T*>(spu.get()));
    // incorrect, need to make refcount same
};
```

- Similarly
  - `const_pointer_cast`
  - `static_pointer_cast`

## Note

- `shared_ptr` NOT solution to everything
- slightly larger size
- meant for _ACTUAL_ shared ownership problems