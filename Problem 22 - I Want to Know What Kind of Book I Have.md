# November 1, 2018

# Problem 22 - I want to know what kind of book I have?

- For simplicity, we assume our initial hierarchy for `Book`

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

## C-style casting

```cpp
(type) expr;
```

- forces the expression to be treated as type `type`

```cpp
int addr = ...;
int *p = (int*) addr;
```



## C++ casting operators

- breaks type system
  - something not this type is now treated as this type
- 4 operators



### `static_cast`

   - for conversions with a well-defined semantics

   ```cpp
   void f(int a);
   void f(double b);
   int x;
   f(static_cast<double>(x));
   ```

   - super class ptrs to subclass ptr

   ```cpp
   Book *b = new Text {...};
   Text *t = static_cast<Text *>(b); // have to know b points at text, else undefined behavior
   ```



### `reinterpret_cast`

   - casts without a well defined semantic
   - unsafe
   - implementation dependent

```cpp
Book *b = new Book{...};
int *p = reinterpret_cast<int*>(b);
```

- mutate private field of struct by creating similar struct and `reinterpret_cast`ing



### `const_cast`

- adding and removing `const`
- the only C++ cast that can cast away `const`

```cpp
void g(Book &b);
void f(const Book &b) {
    g(b); // illegal
}

void h(const Book &b) {
    g(const_cast<Book &>(b)); // perfectly fine
}
```

- we can divide the program into `const` and non-`const` parts which are individually self-consistent
- `const` poisoning at edge / boundary
  - bad way to fix is `const_cast`



### `dynamic_cast`

```cpp
Book *pb = ...;
```

- what if we do not know whether pb points at a Text?
- `static_cast` is __NOT__ safe here

```cpp
Text *pt = dynamic_cast<Text *>(pb);
```

- if `pb` is a Text, or subclass of Text, cast succeeds
- else `pt` is  null pointer

```cpp
if (pt) {...pt->getTopic()...;}
else ...; // not a Test
```

#### ie - type fork

```cpp
void whatIsIt(Book *pb) {
    if(dynamic_cast<Text *>(pb)) cout << "Text";
    else if(dynamic_cast<Comic *>(pb)) cout << "Comic";
    else cout << "Book";
}
```

- in general, not a good indicator of good style
- what happens when we create a new `Book` type

#### ie Dynamic reference casting

```cpp
Book *pb = ...;
Text &t = dynamic_cast<Text &>(*pb);
```

- if `*pb` is a `Text` - ok
- if not, raises `exception`
  - `std::bad_cast`

#### Note

- dynamic casting works by accessing an object's Run-Time Type Information (RTTI)
  - stored in the vtable for the class
- you can only use `dynamic_cast` on objects with at least one `virtual` method
  - no vtable
    - no place for dynamic table to work

## Dynamic Reference Casting

- offers a possible solution to the polymorphic assignment problem
  - before, we made super class pure virtual

```cpp
Text &Text::operator=(const Book &other) {
    const Text &textother = dynamic_cast<const Text &b>(other); // throws if other is not a text
    if(this == &other) return *this;
    topic = other.topic;
    return *this;
}
```

# Recursive Descent Parsing

```cpp
#ifndef __BOOL2_H_
#define __BOOL2_H_

#include <string>
using namespace std;

class expression {
    public:
    virtual string pretty_print() = 0;
    virtual boo evaluate() = 0;
    virtual void set(string name, bool val) = 0;
};

class or_expression: public expression {
    unique_ptr<expression> left, right;
    public:
    or_expression(unique_ptr<expression> left, unique_ptr<expression> right):
    left{std::move(left)}, right{std::move(right)} {}
    string pretty_print() override {
        return "Or (" + left->pretty_print() + "," + right->pretty_print() + ")";
    }
    bool evaluate() override {
        return left->evaluate() || right->evaluate();
    }
    void set(string name, bool val) {
        left->set(name, val);
        right->set(name, val);
    }
}

class and_expression: public expression {
    unique_ptr<expression> left, right;
    public:
    and_expression(unique_ptr<expression> left, unique_ptr<expression> right):
    left{std::move(left)}, right{std::move(right)} {}
    string pretty_print() override {
        return "And (" + left->pretty_print() + "," + right->pretty_print() + ")";
    }
    bool evaluate() override {
        return left->evaluate() && right->evaluate();
    }
    void set(string name, bool val) {
        left->set(name, val);
        right->set(name, val);
    }
}

class variable: public expression {
    string name; bool val;
    public:
    variable(string name, bool val=false):
    name{name}, val{val} {}
    string pretty_print() override {
        return name + "=" + to_string(val);
    }
    bool evaluate() override {return val;}
    void set(string name_, bool val_) {
        if(name == name_) {val = val_;} 
    }
};

unique_ptr<expression> parse_variable(const string &s) {
    return make_unique<variable>(s);
}

unique_ptr<expression> parse_and(const string &s) {
    size_t i = s.find('&');
    if(i == string::npos) return parse_variable(s);
    string s1 = s.substr(0, i);
    string s2 = s.substr(i+1);
    
    return make_unique<and_expression>(parse_variable(s1), parse_and(s2));
}

unique_ptr<expression> parse_or(const string &s) {
    size_t i = s.find('|');
    if(i == string::npos) return parse_and(s);
    string s1 = s.substr(0, i);
    string s2 = s.substr(i+1);
    
    return make_unique<or_expression>(parse_and(s1), parse_or(s2));
}

#endif
```