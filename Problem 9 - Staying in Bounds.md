# October 10, 2018

# Problem 9: Staying in Bounds

## Consider

```cpp
vector v;
v.push_back(2);
v.push_back(3);
v[2]; // undefined behavior, may/may not crash
```

## Can we make this safer?

- second, safer fetch operation
- when invalid access:
  - returning some int?
  - returning non-int?
  - infinite loop? :)
  - crash the program?

```cpp
class vector {
    ...;
    public:
    ...;
    int &at(size_t i) {
        if(i < n) { // note size_t unsigned :)
            return a[i];
        }
        else {
            
        }
    }
    ...;
}
```

## Raise an exception

```cpp
class range_error {};
class vector {
    ...;
    public:
    int &at(size_t i) {
        if(i < n) {...;}
        else throw range_error {};
        // construct an object of type range_error and "throw it"
    }
}
```

## Client

### Do Nothing

```cpp
vector v;
v.push_back(1);
v.at(1); // execution will crash the program
```

### Catch It

```cpp
try {
    vector v;
    v.push_back(1);
    v.at(1);
}
catch(range_error &r) { // r is thrown object, taken by reference => only 1 copy
    // do something
}
```

### Duck - let caller catch error

``` cpp
int f() {
    vector v;
    v.push_back(1);
    v.at(1);
    
    return 0;
}

int g() {
    try {
        f();
    }
    catch(range_error &r) {
        // do something
    }
    
    return 0;
}
```

- execution will propagate back through the call chain until a handler is found
  - unwinding the stack
- if no handler found, program aborts
- control resumes after the catch block (problem code is not retried)

## What happens when a ctor throws an exception?

- object considered partially constructed
- destructor not going to run on the objects

```cpp
class C {...};
class D {
    C a;
    C b;
    int *c;
    
    public:
    D() {
        c = new int[10];
        ...;
        if(...) throw ...; // *
        ~D() { delete [] c;}
    }
};
D d;

// after *, D object is not fully constructed so ~D() will not on d
// But a, b fully constructed, so their dtors will run
```

- So if a ctor wants to throw, it must clean up after itself

```cpp
D() {
    c = new int[10];
    ...;
    if(...) {
        delete[] c;
        throw ...;
    }
}
```

## What happens when a dtor throws?

- trouble
- by default, program aborts immediately
  - std::terminate

```cpp
class myExn {};
class C {
    ...;
    ~C() noexcept(false) {
        throw myExn {};
    }
};
void h() {C c1;} // dtor for c1 throws at the end of h
void g() {
    C c2; 
    h(); // throws, so unwind through g, but then c2 dtor throws as we leave g
}
void f() {
    try {g();} // now 2 unhandled exceptions => defined: guaranteed immediate termination :(
    catch (myExn &e) {
        ...;
    }
}
```

- Never let dtors throw, swallow the exns if necessary

## NOTE:

- able to throw any value, not just objects