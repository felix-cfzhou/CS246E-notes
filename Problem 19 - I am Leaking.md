# October 30, 2018

# Problem 19 - I'm Leaking

## Consider

```cpp
class X {
    int *a;
    public:
    X(int n): a{new int[n]} {}
    ~X() {delete[] a;}
};

class Y: public X {
    int *b;
    public:
    Y(int n, int m): X{n}, b{new int[m]} {}
    ~Y() {delete[] b;}
}
```

### Note:

- `Y`'s destructor will call `X`'s destructor (Step 3)

```cpp
X *px = new Y{3, 4};
delete px; // Leaks, calls X's dtor but not Y's
```

### Recall:

cpp decides which destructor which runs depending on pointer type by DEFAULT

- the destructor should be __`virtual`__

```cpp
class X {
    ...;
    public:
    ...;
    virtual ~X() {delete[] a;} // no more leak
}
```

### Note:

- __Always__ make the destructor virtual in classes that are meant to be superclasses
  - even if the destructor does nothing!
  - we never know what the subclass might do!
    - need to make sure its destructor gets called
  - Also, always give your virtual destructor an implementation
    - __even__ if empty!
      - will get called by the subclass destructor
- If a class is not meant to be a superclass
  - there should not be any `virtual` methods
  - no need to incur cost of `virtual` methods needlessly
  - leave the destructor non-virtual, but declare class `final`

```cpp
class X final { // keyword if occurs in this particular spot (and override)
    ...; // cannot be subclassed
}
```