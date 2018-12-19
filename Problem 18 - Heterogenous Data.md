# Problem 18 - Heterogeneous Data

I want a mixture of types in my vector

- cannot do this with a template

ie

```cpp
vector<template<typename T> T> v; // please do not do this
```

- not allowed

- templates are compile time entity
  - do not exist at run time

ie

- fields of a struct

```cpp
class MediaPlayer {
    template<typename T> nowPlaying; // songs, movies, etc
}
```

## C solution

```cpp
union Media {Song s; Movie m;};
Media nowPlaying;
```
### `void*`
```cpp
void *nowPlaying; // ptr to anything :(
```

## Note

Containers in a collection will usually have something in common

- provide common interface

Can be viewed as different "kinds" of a more general "thing"

### Standard Example

```cpp
class Book { // super class, base class
    String title, author;
    int length;
    public:
    Book(string title, string author, int length): ... {}
    bool isHeavy() const {return length > 100;}
    string getTitle() const {return title;}
    // etc.
}

class Text: public Book {
    string topic;
    public:
    Text(string title, string author, int length, string topic):
    Book{title, author, length}, topic{topic} {}
}
```

# Recall

```cpp
class Book { // super class, base class
    String title, author;
    int length;
    public:
    Book(string title, string author, int length): ___ {}
    bool isHeavy() const {return length > 100;}
    string getTitle() const {return title;}
    // etc.
}

class Text: public Book {
    string topic;
    public:
    Text(string title, string author, int length, string topic):
    Book{title, author, length}, topic{topic} {}
}
```

# October 25, 2018

```cpp
class Book { // super class, base class
    String title, author;
    int length;
    public:
    Book(string title, string author, int length): ___ {}
    bool isHeavy() const {return length > 100;}
    string getTitle() const {return title;}
    // etc.
};

class Text: public Book {
    string topic;
    public:
    Text(string title, string author, int length, string topic):
    Book{title, author, length}, topic{topic} {}
    bool isHeavy() const {return length > 200;}
    String getTopic() {return topic;}
};

class Comic: public Book {
    string hero;
    public:
    Comic(...) {} // exercise
    bool isHeavy() const {return length > 50;}
    string getHero() const;
};
```

## Sub classes

- inherit from superclass
  - fields
  - methods
- all three classes have
  - title
  - author
  - lengths
  - `getTitle()`
  - `getAuthor`
  - `getLength()`
  - `isHeavy()`

## ... Except the above does not work

```cpp
string Text::isHeavy() const {return length > 200;}
```

- `length` is a private field

### Options

1.

```cpp
class Book {
    string title, author;
    protected:
    int length;
    public:
    ...;
};
```

## Protected

- accessible only to this class and its subclasses

2.

```cpp
string Text::isHeavy() const {return getLength() > 200;}
```

- call public method

## Recommended

- option 2
- no control over what subclass will do
- when we make field `protected`, no guarantee that subclasses will respect invariants
- `protected` weakens encapsulation
  - cannot enforce invariants

to have subclasses with privileged access

- keep fields private
- provide protected `get___` and `set___` methods

# Updated Object Creation / Destruction Steps

## Creation

1. Space allocated
2. Super class constructed
3. Fields constructed
4. Constructor body

## Destruction

1. dtor body
2. Fields destructed
3. Super class destructed
4. Space deallocated

## Revisit Everything

- type compatibility
  - `Texts` and `Comics` are special kinds of Books
- should be usable in place of Book

```cpp
Book b = Comic{" ", " ", 75, " "};

b.isHeavy(); // false
```

- `b` is a book
  - b is __NOT__ a comic
- Consequence that c++ has stack allocated objects
- `Book b` sets aside enough space to hold a book
  - `Comic` part is chopped off
  - __"slicing"__
    - id `Book b` has no Hero field :(

```cpp
Book::isHeavy() // runs
```

### Slicing

- happens if super class and super class are the exact same 

```cpp
vector<Book> library;
library.push_back(Comic {...});
```

- only Book parts will be pushed
- NOT a heterogeneous collection

```cpp
void f(Book books[]); // raw arrays
Comic comics[] {...};
f(comics); // is legal!, but never do this
```

- size of `Comic` object not same size as `Book`
  - misalignment of array

### Note

- slicing does __NOT__ happen with pointers

```cpp
Boook *bp = new Comic{..., ..., 75, ...}; // perfectly legal

bp->isHeavy(); // still false :(
```

## Rule

- the choice of which `isHeavy` to run is dependent on _pointer_ (__static type__)
  - NOT the object (__dynamic type__)

### Why?

- cheaper

### C++ Design Principle

"If you do not use it, you should not have to pay for it"

- if we want more expensive operation, we must ask for it

## Make `*bp` act like a `Comic` when it is a `Comic`:

```cpp
class Book {
    ...;
    public:
    ...;
    virtual bool isHeavy() const {...;}
}

class Comic: public Book {
    ...;
    public:
    ...;
    bool isHeavy() const override {...;}
}
```
- note that declaring virtual makes function always virtual

```cpp
bp->isHeavy(); // now works are desired
```

### Note

- `override` simply checks we are indeed overriding a function

## True Heterogeneous Collection

```cpp
vector<Book*> library;
library.push_back(new Book{.......});
library.push_back(new Comic{.......});

// even better?
vector<unique_ptr<Book>> library;
```



```cpp
in howManyHeavy(const vector<Book *> &v) {
    int count=0;
    for(auto &b: v) {
        if(b->isHeavy()) ++count;
    }
    
    return count;
}
...;
for(auto &b : library) delete b; // can be avoided by using unique_ptrs
```

### Note

- correct version of isHeavy is always chosen
  - even though we do not know what is in the vector (items not same type)
  - ___POLYMORPHISM___

## How do `virtual` methods "work"
- and why are they more "expensive"
- implementation dependent, but _nearly_ universal

every `Book` object has virtual pointer to a virtual table which has pointers to implementations of `isHeavy`
- virtual table is ___only___ for virtual functions

### Non-virtual methods
- turns into ordinary function calls


### At least 1 virtual method
- compiler creates a table of function pointers, one per class, "vtable"
- each object contains a pointer to its class's "vtable"
  - "vptr"
- Calling a method:
  - follow "vptr" to "vtable", to function ptr, to the code

### Note

- "vptr" is normally the first field
- always know where vptr is
- virtual methods incur a cost in time (extra pointer dereference) and in space (each object gets a vptr)

## Closing Remarks

If a subclass does not override a virtual method, its vtable will point at the super class implementation