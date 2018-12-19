# Problem 29 - Polymorphic Cloning

```cpp
Book *p = ...;
Book *pb2 = ...; // I want a copy of *pb
```

- cannot call a ctor directly
  - do not know what *pb is

## Standard Solution: virtual clone method

```cpp
class Book {
    ...;
    public:
    virtual Book *clone() {
        return new Book{*this};
    }
};

class Text: public Book {
    Text *clone() override {
        return new Text{*this};
    }
}

class Comic: public Book {
    Comic *clone() override {
        return new Comic{*this};
    }
}
```

- can we reuse code?

```cpp
class AbstractBook {
    public:
    virtual AbstractBook *clone() = 0;
    virtual ~AbstractBook();
};

template<typename T> class Book_cloneable: public AbstractBook {
    public:
    T *clone() override {return new T{static_cast<T &>(*this)};}
}

class Book: public Book_cloneable<Book> {...};
class Text: public Book_cloneable<Text> {...};
class Text: public Book_cloneable<Comic> {...};
```