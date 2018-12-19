# Problem 8 - Efficient Iteration

Consider:

```cpp
Vector v;
v.push_back(...);
...;
for(size_t i=0; i<v.size(); ++i) {
    cout << v[i] << " "; // O(1)
}

list l;
l.push_front(...);
...;
for(size_t i=0; i<l.size(); ++i) {
    cout << l[i] << " "; // O(i)
}
```

No access to "next" ptrs

- efficient iteration?

# Design Pattern

- well-known solutions to well-studied problems
- adapted to suit current needs

# Iterator Pattern

- efficient iteration over a collection without exposing underlying structures

## Idea

- create a class that "remembers" where you are in the list
  - abstraction of a pointer

## Inspiration: C

```cpp
for(int *p=arr; p!=a+size; ++p) printf("%d\n", *p);
```

```cpp
class list {
    struct Node {...};
    Node *theList;
    
    public:
    ...;
    class iterator {
        Node *p;
        
        public:
        iterator(Node *p): p{p} {}
        boolean operator!=(const iterator &other) const {
            return p != other.p;
        }
        iterator &operator++() {
            p=p->next;
            return *this;
        }
        int &operator*() {
            return p->data;
        }
    };
    ...;
    
    iterator begin() {
        return iterator { theList };
    }
    iterator end() {
        return iterator { nullptr };
    }
};
```

Now

```cpp
list l;
...;
...;
...;
for(list::iterator it=l.begin(); it!=l.end(); ++it) { // O(n)
    cout << *it << " ";
}
```

# Recall

```cpp
class list {
    struct Node {...};
    Node *theList;
    
    public:
    ...;
    public iterator {
        Node *p;
        
        public:
        iterator(Node *p) p{p} {}
        bool operator!=(const iterator &other) const { return p!=other.p; }
        int &operator*() const { return p->data; }
        iterator &operator++() { p=p->next; return *this; }
    };
    iterator begin() { return iterator {theList}; }
    iterator end() { return iterator {nullptr}; }
}
```

# October 3, 2018

## Should list::begin, list::end be const methods?

### Consider

```cpp
ostream &operator<<(ostream &out, const list &l) {
    for(list::iterator it=l.begin(); it!=l.end(); ++it) {
        out << *it << ' ';
    }
    
    return out;
}
// will not compile if begin and end are not const
```

If they are:

```cpp
ostream &operator<<(ostream &out, const list &l) {
    for(...) {
        out << *it << ' ';
        ++*it; // increments an item in the list
        // compiles but should not as not very const
        // works since *it returns a non-const ref
    }
    
    return out;
}
```

So iteration over const is different from iteration over non-const

```cpp
class list {
    ...;
    public:
    class iterator { // as before
        ...;
        int &operator*() const;
    };
    class const_iterator {
        Node *p;
        
        public:
        const_iterator(Node *p): p{p} {}
        bool operator!=(const const_iterator &other) const;
        const_iterator &operator++();
        
        const int& operator*() const {return p->data;}
    };
    iterator begin() {return iterator {theList}; }
    iterator end() {return iterator {nullptr};}
    
    const_iterator begin() const {return const_iterator {theList}; }
    const_iterator end() const {return const_iterator {nullptr}; }
}

ostream &operator<<(ostream &out, const list &l) {
    for(list::const_iterator it=l.begin(); it!=end(); ++it) {
        out << *it << ' ';
    }
    
    return out; // works
}
```

### Auto:

```cpp
auto x = expr;
// saves writing x's type
// x is given same type as expression

// in c, auto means stack allocated
```

```cpp
// shorter
ostream &operator<<(ostream &out, const list &l) {
    for(auto it=l.begin(); it!=l.end(); ++it) {
        ...; // as before
    }
}

// even shorter
ostream &operator<<(ostream&out, const list &l) {
    for(auto n : l) out << n << ' '; // range based for loop
    
    return out;
}
```

### Range-based for loops

- require methods begin/end that return an iterator object
- the iterator class must support unary*, prefix++, !=

#### Note:

```cpp
for(auto n : l) ++n; // n is declared by value, so ++n increments n, NOT the list items

for(auto &n : l) ++n; // n is reference, will update the list items

for(const auto &n : l); // n const ref (no copy), cannot be modified

l; // expression evaluated once
```

## One small encapsulation problem...

### client:

```cpp
list::iterator it {nullptr}; // example of forgery, client can create "end" iterator w/o l.end();
```

### To fix:

- make iterator ctors private
  - note list can no longer create iterators either
- give list special permission
  - friendship

## Friendship

- in general NOT symmetric
- Limit friendships!!
  - more friendships => weaker encapsulation effect
  - ripple effect when implementation changes
- Only make friends when smt can do smt for u :)

```cpp
class list {
    ...;
    public:
    class iterator {
        ...;
        iterator(Node *p); // Now private
        
        public:
        ...;
        friend class list; // list can call ctor, can be put anywhere
    }
    class const_iterator {
        ...;
        const iterator(Node *p); // Now private
        
        public:
        ...;
        friend class list; // list can call ctor, can be put anywhere
    }
}
```

### For vectors

```cpp
class Vector {
    ...;
    public:
    class iterator {...};
    class const_iterator {...};
    ...;
}

// OR
typedef int* iterator;
typedef const int* const_iterators;
```