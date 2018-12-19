# Problem 32 - Total Control over Vectors and Lists

- incorporating custom allocators into containers
- make the allocator and arg to the template
  - with a default value

```cpp
template<typename T, typename Allocator<T>> class vector{...};

template<typename T> class allocator {
    public:
    // big 5 (no fields)
    T *address (T &x) {return &x;}
    T *allocate(size_t n) {return ::operator new(n*sizeof(T));}
    void deallocate(T *p) {::operator delete(p);}
    void construct, destroy (placement versions); // pseudo code
}
```

- Lists are not so easy
  - same alloc param
  - want to allocate Nodes, not T's

# Recall

```cpp
template<typename T, typename Alloc=allocator<T>> class list {...}
```

- lists are node based
  - we wish to allocate nodes not T's

# November 29, 2018

## Convention

```cpp
template<typename U> struct rebind {
    using other = allocator<U>;
}

template<typename T, typename Alloc=allocator<T>> class list {
    typename Alloc::rebind<Node>::other alloc;
    // ... then as before
}
```