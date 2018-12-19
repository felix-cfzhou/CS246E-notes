# Problem 27: Collecting Stats

- I want to know how many students I create:

```cpp
class Student {
    int assns, mt, final;
    static int count; // associated with the class NOT individual objects
    
    public:
    // accessors
    int getCount() {return count;} // more like scoped functions, no "this" param
    Student(...): ... {...; ++cout;}
}

// .cc file
int Student::count = 0; // must define var in .cc
Student s1{...}, s2{...}, s3{...};
cout << Student::getCount(); // 3
```

## I want to count objects in other classes

- how can we abstract this solution into reusable code?

```cpp
template<typename T> struct count {
    static int count;
    Count() {++count;}
    count (const Count &) {++count;}
    Count (Count &&) {++count;}
    ~Count() {--count;}
    static int getCount() {return count;}
};

template<typename T> int Count<T>::count = 0;
```

```cpp
class Student: Count<Students> { // private inheritance
    int assns, mt, final;
    
    public:
    Student(...): ... {...}
    // accessors
    
    using Count::getCount; // bring Count::getCount visible
};

class Book: Count<Book> {
    public:
    using Count::getCount;
}
```

- inherits Count's impl without creating an is-a relationship
  - the "inheritance" has been kept private
- members of `Count` become private in Student

## Why is Count a template?

- if not, every private inheritance of Count would use the same variable
- inheriting from template specialized for own class not uncommon

## The Curiously Recurring Template Pattern (CRTP)