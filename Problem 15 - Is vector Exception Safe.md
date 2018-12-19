# Problem 15: Is Vector Exception Safe?

## Consider:

```cpp
template<typename T> class vector {
    size_t n, cap;
    T *theVector;
    
    public:
    vector(size_t n, const T &x):
    n{n},
    cap{n},
    theVector{static_cast<T*>(operator new(n * sizeof(T)))}
    {
        for(size_t i=0; i<n; ++i) new (theVector+i) T(x); // copy ctor
    }
}
```

- partially constructed vector if copy ctor throws
  - dtor will not run
  - broken invariant
  - theVector does not contain n valid objects
- if operator new throws, nothing has been allocated
  - no problem
  - strong guarantee

```cpp
template<typename T> vector<T>::vector(size_t n, const T&n): n{n}, cap{n},
theVector{....} {
    size_t progress = 0;
    try {
        for(size_t i=0; i<n; ++i) {
            new (theVector + i) T(x);
            ++progress;
        }
    }
    catch (...) { // from this point on, stop checking function type arguments, take anything
        for(size_t i=0; i<progress; ++i) theVector[i].~T();
        operator delete(theVector);
        
        throw; // rethrows the error
    }
}
```

Abstract the fill part into its own function:

```cpp
template<typename T> void uninitialized_fill(T *start, T*finish, const T&x) {
    T *p;
    try {
        for(p=start; p!=finish; ++p) {
            new (p) T(x);
        }
    }
    catch (...) { // from this point on, stop checking function type arguments, take anything
        for(T*q=start; q!=p; ++q) q->~T();
        throw; // rethrows the error
    }
}
```

```cpp
template<typename T> vector<T>::vector(size_t n, const T &x): n{n}, cap{n},
theVector{static_cast<T*>(operator new(n * sizeof(T)))} {
    try {
        uninitialized_fill(theVector, theVector+n, x);
    }
    catch(...) {
        operator delete(theVector); // can clean this up by using RAII on the array
        throw;
    }
}
```

```cpp
template<typename T> struct vector_base {
    size_t n, cap;
    T *v;
    vector_base(size_t n): n{n}, cap{n}, v{static_cast<T*>(operator new(n * sizeof(T)))} {}
    ~vector_base() {operator delete(v);}
}
```

```cpp
template<typename T> class vector {
    vector_base<T> vb; // cleaned up implicitly when vector is destroyed
    public:
    vector(size_t n, const T &x): vb{n} {
        uninitialized_fill(vb.v, vb.v+vb.n, x);
    }
    ~vector() {
        destory_elements();
    }
    void destroy_elements() {
        for(T *p=vb.v; p!=vb.v+vb.n; ++p) p->~T();
    }
}
```

- we now have strong guarantees for this particular ctor

### Copy Ctor

```cpp
template<typename T> vector<T>::vector(const vector &other): vb{other.size()} {
    uninitialized_copy(other.begin(), other.end(), vb.v); // implementation is trivial and left as an exercise to the reader
}
```

### Assignment

- copy and swap is exception safe
  - as long as swap is nothrow

 ### push_back:

```cpp
template<typename T> void vector<T>::vector::push_back(const T &x) {
    increaseCap();
    new(vb.v + vb.n) T(x); // assume T does the right thing if T throws :)
    // have same vector as before (to client) - good!
    ++vb.n; // do not increment n before certain that construction has succeeded
}
```

### What if increaseCap throws?

```cpp
void increaseCap() {
    if(vb.n == vb.cap) {
        vector_base vb2 {2 * vb.cap}; // RAII
        uninitialized_copy(vb.v, vb.v+vb.n, vb2.v);
        vb2.n = vb.n;
        destroy_elements();
        std::swap(vb, vb2); // swap is no throw on ints and pointers
    }
}
```

#### Note

- only try block employed are in uninitialized_fill and uninitialized copy

### Efficiency

- copying from the old array to the new one
  - moving would be better
  - but moving destroys the old array!
  - if exception thrown during move
    - vector is corrupted :(
- we can only move if certain move operation is nothrow

# Recall

```cpp
void increaseCap() {
    if(vb.n == vb.cap) {
        vector_base vbs {2*vb.cap};
        uninitialized_copy(vb.v, vb.n, vb2.n); // replace with uninitialized_copy_or_move
        vb2.n = vb.n;
        destroy_elements();
        std::swap(vb, vb2);
    }
}
```

- efficiency
  - copy is slow
  - moving is faster
- moving destroys old array
  - if exception is thrown, vector corrupted
  - can only move if guarantee move operation is no throw

# October 23, 2018

# Efficiency

```cpp
template<typename T> void uninitialized_copy_or_move(T *start, T *finish, T *target) {
    T *p;
    try {
        for(p=start; p!=finish; ++p, ++target) {
            new(p) T(std::move_if_noexcept(*target));
        }
    }
    catch (...) { // will never happen if T has a non-throwing move
        for(T *q=start; q!=p; ++q) q->~T();
        throw;
    }
}
```

```cpp
std::move_if_noexcept(x);
```

- produces `std::move(x)`if x has non-throwing move ctor
- produces `x` otherwise

## noexcept

```cpp
class C {
    public:
    C(C &&other) noexcept;
    ...;
}
```

`noexcept`

- if exception thrown in noexcept function
  - `std::terminate`
- move and swap SHOULD be non-throwing :(
  - declare them so
  - allow more optimized code to run

# Is `std::swap` `noexcept`? 
```cpp
template<typename T> void swap(T &a, T &b) {
    T c(std::move(a));
    a = std::move(b);
    b = std::move(c);
}
```

- only `noexcept` if `T` has a `noexcept` move ctor and move assignment

## Conditional `noexcept`

```cpp
template<typename T> void swap(T &a, T&b) noexcept(std::is_no_throw_move_constructible<T>::value && std::is_nothrow_move_assignable<T>::value) {
    ...;
}
```

### Note:

`noexcept` means `noexcept(true)`