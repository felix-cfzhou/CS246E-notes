# Problem 17 - Abstraction over containers

## Recall

`map` from Racket

```scheme
(map f (list a b c)) -> (list (f a)(f b)(f c))
```

- do the same for vector?

### Assumptions

- target has enough space to hold as much of sources as we want to send

```cpp
template<typename T1, typename T2> void transform(
    const vector<T1> &source,
    vector<T2> &target,
    T2 (*f)(T1)
) {
    auto it = target.begin();
    for(auto &x: source) {
        *it = f(x);
        ++it;
    }
} 
```

- OK, but......
  - what if we only want part of source?
  - only send to middle of target, not beginning?
- use iterators

```cpp
template<typename T1, typename T2> void transform(
    vector<T1>::iterator start,
    vector<T1>::iterator finish,
    vector<T2>::iterator target,
    T2 (*f)(T1)
) {
    while(start!=finish) {
        *target = f(*start);
        ++target;
        ++start;
    }
}
```

## Note

does not compile :(

- BUT
  - if we want to transform a list
  - write same code again :(
- what if we want to transform list to a vector (or vice versa)
  - make type variables stand for iterators, not container elements
- But then how to indicate type of f?
  - third template argument

```cpp
template<typename InIter, typename OutIter, typename Fn> void transform(
    InIter start,
    InIter finish,
    OutIter target,
    Fn f)
{
    while(start != finish) {
        *target = f(*start);
        ++target;
        ++start;
    }
}
```

### Note

works over vector iterators, list iterators, or any other kinds of iterators

`InIter`, `OutIter`, can be any types that support `++`, `!=`, `*`, INCLUDING ordinary midterms

# Recall

```cpp
template<typename InIter, typename OutIter, typename Fn> void transform(
    InIter start,
    InIter finish,
    OutIter target,
    Fn f)
{
    while(start != finish) {
        *target = f(*start);
        ++target;
        ++start;
    }
}
```

## Full code for insert

```cpp
template<typename T> class vector {
    iterator insert(interator posn, const T &x) {
        ptrdiff_t offset = posn - begin();
        increaseCap(); // if increaseCap called, posn not valid, so we use precalculated offset <3
        iterator newPosn = begin() + offset;
        new(end()) T (std::move_if_noexcept(*(end()-1)));
        // not yet constructed, needs to be placement newed
        ++vb.n;
        for(iterator it=end()-1; it!=newPosn; --it) {
            *it = std::move_if_noexcept(*(it-1));
        }
        *newPosn = T(x); // &&x similar
        return newPosn;
    }
}
```

# October 24, 2018

## Note

`cpp` will instantiate transform function with any type that has the operations used by the function

### What about the argument type?

- `Fn` can be any type that supports function application

```cpp
class Plus {
    int n;
    public:
    Plus(int n): n{n} {}
    int operator() (int n) {return n+m}
}

Plus p(1);
cout << p(2); // 3 - function object
```

```cpp
transform(v.begin(), v.end(), w.end(), Plus{1});
```

#### OR

```cpp
transform(v.begin(), v.end(), w.begin(), [](int n){return n+1;}) // "lambda"
```

## Lambda

```cpp
[__"capture list"_](__"param list"___) mutable? noexcept? {___body___;}
```

### Semantics

```cpp
void f(T1 a, T2 b) {
    [a, &b](int x){return body;}(args)
}
```

#### translates to

```cpp
void f(T1 a, T2 b) {
    class ~~~ { // class with unknown name
        T1 a;
        T2 &b;
        public:
        ...;
        ~~~ (T1 a, T2 &b): a{a}, b{b} {}
        auto operator(int x) const {return body;}  
    };
    ~~~{a, b}.operator()(args);
}
```

### Mutable lambda

If declared as `mutable`

- `const` object, `mutable` allows changing specific fields
- in lambda case, function is by default `const`
  - if declared `mutable`, `operator()` is then __NOT__ const
  - capture list
    - provides access to selected vars in the enclosing scope

### Note

can only store `lambda` in a variable if we use `auto`