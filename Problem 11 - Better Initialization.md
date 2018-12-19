# Problem 11: Better initialization

long sequence of push_backs is clunky

Array: 

```cpp
int a[] = {1, 2, 3, 4, 5};
vector<int> v;
v.push_back(1);
...;
```

# Recall

```cpp
int a[] = {1, 2, 3, 4, 5};

vector<int> v;
v.push_back(1);
...;

// Goal is better initialization
```

# October 11, 2018

```cpp
template<typename T> class vector {
    ...;
    public:
    vector() {...}
    vector(size_t n, T i = T {}): n{n}, cap{n==0 ? 1 : n}, theVector{new T[cap]} {
     	// NOTE: T{} means 0 if T is a built-in type
        for(size_t j=0; j<n; ++j) theVector[j] = i;
    }
    ...;
    
    vector<int> v; // empty
    vector<int> v {5}; // 0 0 0 0 0
    vector<int> v {3, 5};
};
```

```cpp
#include <initializer_list>

vector<typename T> class vector {
    ...;
    public:
    ...;
    vector() {...}
    vector(size_t n, T i = T{}): ...;
    vector(std::initializer_list<T> init): n{init.size()}, cap{n==0 ? 1 : n}, theVector{new T[cap]}  	{
    	size_t i=0;
        for(const auto t : init) theVector[i++] = t; // try to be careful with references here
    }
};

vector<int> v {1, 2, 3, 4, 5}; // 1 2 3 4 5
vector<int> v {5}; // 5
vector<int> v {3, 5} // 3 5
```

## RULE

- default ctors take precedence over initializer_list ctors, which take precedence over all other ctors

### Can we still get other ctor to run?

- need round bracket initialization

```cpp
vector<int> v(5); // 0 0 0 0 0
vector<int> v(3, 5); // 5 5 5
```

### Note on cost

- items in an init list are stored in contiguous memory
  - (begin returns ptr)
- We are using one array to build another
  - 2 copies in memory
- intializer_lists meant to be immutable
  - do not modify contents
  - do not use as standalone data structures
- But also note, there is only one allocation in vector, not several
  - no doubling and reallocating
    - as there was with a sequence of push_backs

## In general

- If we know how big the vector will be, we can save reallocation cost by requesting memory up front

```cpp
template<typename T> class vector {
    ...;
    public:
    ...;
    void reserve(size_t newCap) {
        if(cap < newCap) {
            T* newVec = new T[newCap];
            for(size_t i=0; i<n; ++i) newVec[i] = theVector[i];
            
            delete [] theVector;
            theVector = newVec;
            cap = newCap;
        }
    }
}
```

### Exercise

- rewrite vector so that push_back uses reserve instead of increaseCap

```cpp
vector<int> v;
v.reserve(10);
v.push_back(...); // can do 10 push_backs without needing to reallocate
```