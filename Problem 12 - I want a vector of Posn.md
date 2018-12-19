# Problem 12 - I want a vector of Posns

```cpp
struct Posn {
    int x, y;
    Posn(int x, int y): x{x}, y{y} {}
};

int main() {
    vector<Posn> v; // Will not compile
}

// Vector's default ctot
template<typename T> vector<T>::vector(): n{0}, cap{1}, theVector{new T[cap]} {}
// Which T objects will be stored in the array?
```

## NOTE

- c++ always calls a ctor when creating an object

- which ctor gets called?

  - default ctor

  - ```cpp
    T {};
    ```

  - Posn does not have one

## How can we create a vector of Posns?

- add default ctor to Posn?
  - caution against
- we need to seperate memory allocation, (object creation step 1), from object initialization, (object creation step 2-4)

### Allocation

```cpp
void *operator new(size_t) {}
```

- allocates size_t bytes
  - no initialization
- returns a void*

#### NOTE:

- in C, void* implicitly converts to any ptr type
- in C++, the conversion requires a cast

### Initialization: "placement new"

```cpp
new(addr) type; // create an object AT address
```

- constructs a type object at whatever address is
- does NOT allocate memory

```cpp
template<typename T> class vector {
    ...;
    public:
    vector(): n{0}, cap{1}, theVector{static_cast<T*>(operator new(sizeof(T)))} {}
    vector(size_t n, T x):
    	n{n},
    	cap{n==0 ? 1 : n},
    	theVector{static_cast<T*>(operator new(cap*sizeof(T)))}
    {
        for(size_t i=0; i<n; ++i) {
            new(theVector+i) T(x);
        }
    }
    ...;
    void push_back(T x) {
        increaseCap();
        new (theVector + (n++)) T(x);
    }
    
    void pop_back() {
        if(n) {
            theVector[--n].~T(); // must explicitly invoke dtor
        }
    }
    
    ~vector() {
        destroy_items();
        operator delete(theVector);
    }
    
    void destroy_items() {
        for(T *p=theVector; p!=theVector+n; ++p) p->~T();
        n=0;
    }
}
```