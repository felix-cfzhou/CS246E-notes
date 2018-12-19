# Problem 13: Less copying!

Before

```cpp
void push_back(int n);
```

Now

```cpp
void push_back(T x) { // if T is object, how many times is T copied?
    increaseCap();
    new(theVector + n++) T(x);
}
```

## Note

if arg is lvalue, 2 copies want 1

1. (T x) is copy ctor
2. T (x) is copy ctor

if arg is rvalue, 1 copy, want none

1. (T x) is move ctor
2. T (x) is copy ctor

## FIX:

```cpp
void push_back(T x) {
    increaseCap();
    new(theVector + n++) T {std::move(x)}
}
```

### Now:

if arg is lvalue, 1 copy

1. (T x) is copy ctor
2. T (x) is move ctor

if arg is rvalue, no copies

1. (T x) is move ctor
2. T (x) is move ctor

# Recall

```cpp
void push_back(T x) {
    increaseCap();
    new (theVector + n++) T (std::move(x));
}
```

- lvalue: copy and move
- rvalue: move and move

# October 17, 2018

# Problem 13 continued

## If T does not have a move ctor:

- 2 copies

### Better:

```cpp
void push_back(const T &x) { // no copy
    increaseCap();
    new (theVector + n++) T(x); // copy ctor
}

void push_back(T && x) { // no copy
    increaseCap();
    new (theVector + n++) T(std::move(x)); // move ctor
}
```

- If T has no move ctor, one copy

## Consider:

```cpp
vector <Posn> v;
v.push_back(Posn {3, 4})
```

1. ctor call for Posn to create the object
2. copy / move construction into vector depending on whether Posn has copy / move ctor
3. dtor on temp object

### Could we eliminate step 1 and 3

- get vector to create object instead of client
- pass ctor args to the vector NOT the actual object

## Note on template functions:

consider:

```cpp
std::swap
```

- works on all types

### Implementation

```cpp
template<typename T> void swap(T &a, T &b) {
    T temp(std::move(a));
    a = std::move(b);
    b = std::move(temp);
}
```

```cpp
int x=1, y=2;
swap(x, y); // Equiv swap<int>(x, y);
```

- no need to say swap(int)
- c++ can deduce this from types of x and y
- in general only have to say `f<T>(...)` if T cannot be deduced from args
- type deduction for template args follow the same rules as type deductions for auto
- Back to vectors - passing ctor args
  - we do not know what types ctor args should have
  - T could be any class, several ctors

## Idea

member template function

- like swap, could take anything
- How many ctor args?

### Solution: variadic templates

```cpp
template<typename T> class Vector {
    ...;
    public:
    ...;
    template<typename... Args> void emplace_back(Args... args) {
        increaseCap();
        new(theVector + n++) T(args...);
    }
}
```

Args is a __sequence__ of type variables denoting the types of the actual arguments of emplace_back

args is a __sequence__ of program variables denoting the actual args of emplace_back

```cpp
vector<Posn> v;
v.emplace_back(3, 4)
```

## Problem:

args is taken by value

can we take args by reference?

- lvalue or rvalue reference?

```cpp
template<typename... Args> void emplace_back(Args &&... args) {
    
}
```

Special Rule:

```cpp
Args && // universal reference, l OR r value reference
```

- points to an lvalue OR an rvalue

## When is a reference universal?

- must have form T&&, where T is the type arg being deduced for current template function call

```cpp
template<typename T> class C {
    public:
    template<typename U> void f(U &&x); // universal
    template<typename U> void g(const U &&x); // NOT universal
    void h (T &&x); // NOT universal, T is NOT being deduced as of now, already deduced
}
```

## Recall:

```cpp
class C {...}
void f(c &&x) {
    g(x); // rvalue ref, x points to rvalue, but IS an lvalue
}
```

# Recall

```cpp
template<typename... Args> void emplace_back(Args&&... args) { // universal ref
    increaseCap();
    new(theVector + n++) T(args...);
}

class C {...}

void f(C &&x) { // rvalue ref (points to rvalue), BUT is lvalue
    g(x); // x passed as lvalue to g
}
```

# October 18, 2018

- If we want to preserve the fact that x is an `r-value` ref, so that a "move" version of g is called (assuming there is one)

```cpp
void f(C &&x) {
    g(std::move(x));
}
```

- but in this case, we do not know if args are lvalues, rvalues or a mix
  - we wish to call move $$\iff$$ the args are rvalues

```cpp
template<typename... Args> void emplace_back(Args&&... args) {
    increaseCap();
    new(theVector + n++) T (std::forward<Args>(args)...);
    // calls std::move if argument is rvalue ref, else does nothing
	// NOW args passed to T's ctor with lvalue/rvalue information preserved
}
```

- __perfect forwarding__