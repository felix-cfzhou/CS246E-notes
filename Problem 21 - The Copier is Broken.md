# Problem 21 - The copier is broken

How do copy and move interact with inheritance?

```cpp
Text::Text(const Text &other): Book{other}, topic{other.topic} {}
```

```cpp
Text::Text(Text &&other): Book{std::move(other)}, topic{std::move(other.topic)} {}
```

## Note:

we moved the `Book` part of `other`, but not the `Text` only fields, so we are gucci

```cpp
Text &Text::operator=(const Text &other) {
    Book::operator=(other);
    topic = other.topic;
    
    return *this;
}
```

## But

```cpp
Book *b1 = new Text {...}, *b2 = new Text {...};
*b1 = *b2;
```

- after the assignment, `b1` has fields from `b2` but __ONLY__ the `Book`parts

### What Happens?

- only the `Book` part is copied
  - __partial assignment__
    - topic does not match the `title` and `author`
      - object is corrupted

#### Solution

```cpp
class Book {
    ...;
    public:
    virtual Book &operator=(const Book &other) {...}
};

class Text {
    ...;
    public:
    Text &operator=(const Book &other) override {...}
    // override keyword makes sure overriding something
    // But can pass in Comic etc YIKES!
}
```

#### But

- must be book or else __NOT__ and override!

## Alternate Solution

- Make all superclasses abstract

```cpp
class AbstractBook {
    string title, author, length;
    protected:
    AbstractBook &operator=(const AbstractBook &other) {...} // non-virtual
    // this is protected and NOT private as subclasses will need it :(
    ...;
    virtual ~AbstractBook() = 0; // to make class abstract
}
AbstractBook::~AbstractBook() {} // must be outside

class Text: public AbstractBook {
    public:
    Text &operator=(const Text &other) {
        Bool::operator=(other); // non-virtual, no mixed assignment
        // not accessible to outsiders
        topic = other.topic;
    }
}

class NormalBook: public AbstractBook {...}
class Comic: public AbstractBook {...}
```

### Note:

- now there cannot be mixed override
- and partial assignment does not compile
- the super class destructor must ALWAYS be implemented, no matter what

## Pure Virtual

- does NOT mean have no implementation
  - still allowed to implement
- means subclass MUST override me
- "reasonable default"
  - in this case do nothing :)