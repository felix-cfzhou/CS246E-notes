# Problem 24 - Abstraction over Iterators

I want to jump ahead `n` spots in my iterator

obvious solution

```cpp
template<typename Iter> Iter advance(Iter it, size_t n) {
    for(size_t i=0; i<n; ++i) ++it;
    return it;
}
```

- Slow
  - $$O(n) $$ time complexity
  - can we just add `n` to it?
    - Depends!
      - for vectors yes
      - for lists no :(
        - `+=` not supported (and if it was, would _still_ be linear time)
    - Related - Can we go backwards
      - vector yes
      - list no
    - So all iterators support `!=`, `*`, `++`
      - but some iterators support other operations
- `list::iterator`
  - calls a __forward iterator__
    - can only go one step forward
- `vector::iterator`
  - __random access iterator__
    - can go anywhere (supports arbitrary ptr arithmetic)
- How can we write `advance` to use `+=` for random access iterators and loop for forward iterators
  - Since we have different types of iterators, create a class hierarchy:
```cpp
struct input_iterator_tag {};
struct output_iterator_tag {};
struct forward_iterator_tag: public input_iterator {};
struct bidirectional_iterator_tag: public forward_iterator_tag {};
struct random_access_iterator_tag: public bidirectional_iterator_tag {};
```

- to associate with a tag
  - inheritance?

```cpp
class List {
    ...;
    public:
    class iterator: public forward_iterator_tag {...};
}
```

- makes it hard to ask what kind of iterator we have
  - cannot `dynamic_cast` as no `virtual` function so no vtable!
  - does not work for iterators that are NOT classes
- Instead
  - make the tag a member

```cpp
class List {
    ...;
    public:
    class iterator {
        ...;
        public:
        typedef forward_iterator_tag iterator_category;
        ...
    }
}
```

- convention that every iterator class will define a type member called `iterator_category`
  - not done yet!
  - template function associating every iterator type with its category

```cpp
template<typename It> struct iterator_traits {
    typedef typename It::iterator_category iterator_category;
};

void transform(vector<T1>::iterator ...);
```

- extra `typename` keyword needed so we know `It::iterator` is a __type__
  - compiler does not know anything about `It`
  - default assumption is that `It` is NOT a types

## Consider

```cpp
template<typename T> void f() {
    T::something x; // only makes sense if T::something is a type
};

// BUT
template<typename T> void f() {
    T::something *x; // is this ptr declaration or a multiplication? :(
}
```

- We cannot know whether `T::something` is a type
  - c++ assumes value until told otherwise

```cpp
template<typename T> void f() {
    typename T::something x;
    typename T::something *y;
}
```

- need to say `typename` whenever we refer to a member type of a template parameter
- ie

```cpp
iterator_traits<List<T>::iterator>::iterator_category;
// forward_iterator_tag
```

specialized version for vector

```cpp
template<typename T> struct iterator_traits<T *> {
    typedef random_access_iterator_tag iterator_category;
};
```

# Recall

```cpp
iterator_traits<List<T>::iterator>::iterator_category;
// forward_iterator_tag
```

specialized version for vector

```cpp
template<typename T> struct iterator_traits<T *> {
    typedef random_access_iterator_tag iterator_category;
};
```

# November 21, 2018

- what do we do with this?

```cpp
template<typename Iter> Iter advance(Iter it, int n) {
    if(typeid(iterator_traits<Iter>::iterator_category) == typeid(random_access_iterator_tag)) {
        return it += n;
    }
    else if(...) {
        ...
    }
}
```

- does not compile if the iterator is not random access 
  - does not have a `+=` operator 
  - if `+=` will cause a compilation error
    - even though it will never be used
- Also, choice of which implementation to use is being made at run-time when the right choice is known at compile-time
  - solve with __overloading__

```cpp
template<typename Iter> Iter doAdvance(Iter it, int n, random_access_iterator_tag) {
    return it += n;
}

template<typename Iter> Iter doAdvance(Iter it, int n, bidirectional_iterator_tag) {
    if(n>0) for (int i=0; i<n ++i) ++it;
    else if(n<0) for (int i=0; i<-n; ++i) --it;
    
    return it;
}

template<typename Iter> Iter doAdvance(Iter it, int n, forward_iterator_tag) {
    if(n >= 0) {
        for(int i=0; i<n; ++n) ++it;
        return it;
    }
    
    throw SomeError{};
}

template<typename Iter> Iter advance(Iter it, int n) {
    return doAdvance(it, n, iterator_traits<Iter>::iterator_category{} );
}
```

- compiler will now select the fast `doAdvance` for random access iterators, the slow `doAdvance` for bidir iterators and the throwing version for forward iterators
  - choices made at compile time
    - no run-time cost
- We used template instantiations to perform compile-time computation
  - __template metaprogramming__
  - express conditions via overloading, repetition via recursive template instantiation

```cpp
template<int N> struct Fact {
    static const int value = N*Fact<N-1>::value;
};

template<> struct Fact<0> {
    static const int value = 1;
};

int x = Fact<5>::value; // evaluated at compile time
```

## Current Solution

```cpp
constexpr int fact(int n) {
    if(n == 0) return 1;
    else return n*fact(n-1);
}
```

### `constexpr` functions

- evaluate the function at compile-time if `n` is a compile-time constant
- else evaluate at run-time
- must be something that can actually be evaluated at compile-time
- cannot mutate non-global variables