# Problem 6: I want a constant vector

## Say we wish to print a vector:

```cpp
ostream &operator<<(ostream &out, const Vector &v) { // const makes it not compile
    for(size_t i=0; i<v.size(); ++i) {
        out << v.itemAt(i) << " ";
    }
    
    return out;
}
// will not compile
```

- we cannot call itemAt(i) on a const object
- what if these methods change fields?

Since methods do not change fields, declare them as const:

```cpp
struct Vector {
    ...;
    size_t size() const;
    // means these methods will not modify fields => can be called on const objects
    int &itemAt(size_t i) const;
    ...;
}

...;
// labelling must be done in both declaration and implementation
size_t Vector::size() const {
    return n;
}
int &Vector::itemAt(size_t i) const {
    return theVector[i];
}
```

Now the loop above will work

## But:

```cpp
void f(const vector &v) {
    v.itemAt(0) = 4; // this works, but undesirable, as v NOT very const
}
```

v is a const object

- cannot change n, cap, theVector
- CAN change items pointed to by the pointer.

Fix?

```cpp
struct Vector {
    ...;
    const int &itemAt(size_t i) const {return theVector[i]};
};

// Now v.itemAt(0) = 4, will not compile EVEN if v is NOT const
```

Fix again?

```cpp
struct Vector {
    ...;
    const int &itemAt(size_t i) const; // called if object is const
    int &itemAt(size_t i); // called if object it NOT const :)
}

inline const int &Vector::itemAt(size_t i) const { return theVector[i]; }
inline int &Vector::itemAt(size_t i) { return theVector[i]; }
// writing method body in the class implicitly suggests inlining
```

# Recall

```cpp
struct Vector {
    ...;
    const int &itemAt(size_t i) const;
    int &itemAt(size_t i);
};
```

# October 2, 2018

## More Idiomatic

```cpp
struct Vector {
    ...;
    const int &operator[](size_t i) { return theVector[i]; }
    int &operator[](size_t i) { return theVector[i]; }
    // note by putting method body inside class implicitly declares method inline
};
```