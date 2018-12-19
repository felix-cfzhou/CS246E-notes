# Problem 10: I want a vector of chars

- start over? naw

## Templates - major abstraction mechanism

- generalize over types

```cpp
template <typename T> class vector {
    size_t n, cap;
    T* theVector;
    
    public:
    vector();
    ...;
    void push_back(T n);
    T &operator[](size_t i);
    const T &operator[] const(size_t i);
    typedef T *iterator;
    typedef const T *const_iterator;
}

template<typename T> vector<T>::vector(): n{0}, cap{1}, theVector{new T[cap]} {}
template<typename T> void vector<T>::push_back(T n) {...;}
// etc
```

### NOTE:

must put implementation in the .h file

## main.cc

- first time compiler encounters vector\<int\>, create version of vector class where int replaces T
  - compiles new class
  - cannot do that unless it knows all the details
- so implementation must be available, ie in the .h file
- inline is fine

```cpp
int main() {
    vector<int> v; // vector of ints
    v.push_back(1);
    ...;
    vector<char> w; // vector of chars
    v.push_back('a');
    ...;
}
```