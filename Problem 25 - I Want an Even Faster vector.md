# Problem 25 - I want an even faster vector

IN the good old days of C, you would copy an array (even of structs!) very quickly by calling `memcpy` (like `strcpy` but for arbitrary memory not just strings)

`memcpy` was written in assembly, fast as machine could possibly be

In C++

- copies invoke copy ctors, which are function calls `:(`

# Recall

# Problem 25 - I want an even faster vector

In the good old days of C, you would copy an array (even of structs!) very quickly by calling `memcpy` (like `strcpy` but for arbitrary memory not just strings)

`memcpy` was written in assembly, fast as machine could possibly be

In C++

- copies invoke copy ctors, which are function calls `:(`



# November 22, 2018

## POD

In C++, type is considered POD (plain old data) if it has

- trivial default constructor (equiv. = default)
- trivially copiable
  - copies and moves
- dtor has default implementation
- __Standard Layout__
  - no virtual methods or bases
- all members have same visibility
- no reference members
- no fields in both base class and sub class
  - or in multiple base classes

For POD types, semantics are compatible with C, and memcpy is safe to use

```cpp
template<typename T> class vector {
    T *theVector;
    size_t n, cap;
    
    public:
    ...;
    vector(const vector &other): ... {
        if(std::is_pod<T>::value) {
            memcpy(theVector, other.theVector, n*sizeof(T));
        }
        else {
            // as normal
        }
    }
}
```

- works but decision can be made at compile-time

```cpp
template<typename T> class vector {
    ...;
    public:
    template<typename X=T> vector(typename enable_if<std::is_pod<X>::value, const T&>::type other): ... {
        memcpy(theVector, other.theVector, n*sizeof(T));
    }
    
    template<typename X=T> vector(typename enable_if<!std::is_pod<X>::value, const T&>::type other): ... {
        // placement new loop
    }
    ...;
}
```

## `enable_if`

```cpp
template<bool b, typename T> struct enable_if;

template<typename T> struct enable_if<true, T> {
    using type = T;
};
```

- with meta-programming, what you say is equally as important as what you do not
- if `b` is `true`, `enable_if` defines a struct whose 'type' member typedef is T
  - so if `std::is_pod<X>::value = true`, then `enable_if<std::is_pod<X>::value, const T&>::type` $$\rightarrow$$ `const T&` 
- If `b` is `false`, then the struct is declared but not defined
  - so `enable_if<b, T>` will no compile
- So one of these two versions of the copy ctor (the one with the false condition) will not compile
  - how is this valid program?
- C++ Rule: __SFINAE__
  - __Substitution Failure Is Not An Error__

## SFINAE

If `t` is a type

```cpp
template<typename T> .... f (...) {...}
```

is a template function, and substituting `T=t` results in an invalid function, the compiler does __NOT__ signal an error, it simply removes that instantiation from consideration during overload resolution

### NOTE

- if no function is in scope to handle overload call
  - that is an error

### Why is this wrong?

- ie why do we need extra template out front?

```cpp
template<typename T> class vector {
	...;
    public:
    vector(typename enable_if<std::is_pod<T>::value, const T&>::type other): ... {}
    vector(typename enable_if<!std::is_pod<T>::value, const T&>::type other): ... {}
}
```

- SFINAE only works for template functions!!
  - the above are normal ctors and thus __NOT__ templates
- the deduction depends on T, but T's value are determined when we decided what to put in the vector
- if substituting T=t fails, it invalidates the entire vector class
  - not just method
- make the method a separate template
  - with new arg X
    - defaulted to T
  - do `is_pod<X>` not `is_pod<T>`
- compiles but crashes

### Default Behavior

- compiler gives default copy ctor
  - not smart enough to recognize the above are copy ctors
    - shallow copies
  - solution?

```cpp
template<typename T> class vector {
    ...;
    public:
    vector(const vector &other) = delete; // disable?
    ...;
};

// NOT ALLOWED
```

- cannot disable copy ctor and then write one
  - solution?

#### Overloading

- extra parameters, so NOT copy ctor

```cpp
template<typename T> class vector {
    ...;
    struct dummy {};
    public:
    vector(const vector &other): vector{other, dummy{}} {}
    template<typename X=T> vector(typename enable_if<std::is_pod<X>::value, const vector<T>&>::type other, dummy): ... {...}
    template<typename X=T> vector(typename enable_if<!std::is_pod<X>::value, const vector<T>&>::type other, dummy): ... {...}
    ...;
};
```

- overload with dummy argument
- have cpy ctor delegate to others
  - copy ctor inline so no function call overhead
- works `:)`!

## Helper Definitions

make `is_pod`, `enable_if` easier to use

```cpp
template<typename T> constexpr bool is_pod_v = std::is_pod<T>::value;

template<bool b, typename T> using enable_if_t = typename enable_if<b, T>::type;

template<typename T> class vector {
    ...;
    template<typename X=T> vector(enable_if_t<is_pod_v<X>, const vector<T> &> other, dummy): ... {...}
}
```

- we now have machinery to implement `std::move` and `std::forward`

### `std::move`

- at its core, simply a cast

```cpp
template<typename T> T&& move(T &&x) {
    return static_cast<T &&>(x);
}
```

- does not work as `T &&` is universal reference in this case
  - If `T` was an lvalue, stays lvalue `:(`
  - need to make sure T is not an lvalue ref
    - if so, get rid of it

```cpp
template<typename T> inline constexpr ........ move(T &&x) {
    return static_cast<std::remove_reference<T>::type &&>(x);
} // std::remove_reference<T> turns T& or T&& to T
```

- exercise: implement the above

```cpp
template<typename T> struct remove_reference {
    using type = T;
}

template<typename T> struct remove_reference<T &> {
    using type = T;
}

template<typename T> struct remove_reference<T &&> {
    using type = T;
}
```



#### Can we save typing and use auto?

```cpp
template<typename T> auto move(T &&x) {...}
```

- NO!

##### By-value `auto`

```cpp
auto x = y;

int z;
int &y = z;
auto x = y; // x is int

const int &w = z;
auto v = w; // v is int
```

- by-value auto ignores references and outer consts
  - uses the type a value would have in expressions if copied
- by-reference `auto &&` on the other hand is a _universal reference_
  - follows template deduction rules
- need a type deduction rule that does not discard refs

### `decltype`

```cpp
decltype(...);
```

- return the type that ... was declared to have

```cpp
decltype(var);
```

- declared type of var

```cpp
decltype(expr);
```

- returns an lvalue or rvalue ref
  - depends on whether expr is an lvalue or an rvalue

```cpp
int z;
int &y = z;
decltype(y) x = z; // x is int&

x=4; // affects z
decltype(z) s = z; // s is int
s=5; // does not affect z

auto x = z; // x is int
x=4; // does not affect z

decltype((z)) s = z; // s is int&, as (z) is an EXPRESSION
s=6; // affects z;
```

- Now

```cpp
decltype(auto)
```

- perform type deduction like auto but use the decltype rules
  - do not throw away refs

- returning to prev example

```cpp
template<typename T> inline constexpr decltype(auto) move(T &&x) {
    // as before
}
```