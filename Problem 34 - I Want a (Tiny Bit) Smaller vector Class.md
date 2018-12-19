# Problem 34 - I want a (tiny bit) smaller vector class

- vector / vector_base have an allocator field `a`
  - itself has no fields
- what is its size?
  - 0?
  - No - not allowed in C++
- Thus every type has size at least 1
  - compiler insert dummy character
  - our vector is ONE BYTE LARGER `>:(`
- Due to alignment
  - may have lost anywhere from 1 - 8 bytes `>:(`
- If both vector and vector base have allocator fields
  - may have lost twice that much!

## Empty Base Optimization (EBO)

- to save this space
- an empty __base class__ does __NOT__ have to occupy space in an object
  - so we make allocator a parent not a field
- `vector_base` also a super class

```cpp
template<typename T, typename Alloc=std::allocator<T>> struct vector_base: private Alloc {
    // struct has default public inheritance
    size_t n, cap;
    T *v;
    
    using Alloc::allocate;
    using Alloc::deallocate;
    // etc, as we do not know anything about Alloc, must bring members into scope
    ...;
};

template<typename T typename Alloc=std::allocator<T>> class vector: vector_base<T, alloc> {
    // private inheritance since not an "is-a" relationship;
    using vector_base<T, Alloc>::n;
    using vector_base<T, Alloc>::cap;
    using vector_base<T, Alloc>::v;
    ...;
};

// full implementation is exercise
```

# Conclusion

- Not a course on C++
- Or subtype polymorphism

## Goal

- abstraction
- build general things which work and hide ugly details
  - time, space optimizations
- C++ has high-level abstractions as well as low-level control