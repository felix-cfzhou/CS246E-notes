# Problem 5: Moves

Consider:

```cpp
Node plusOne(Node n) {
    for(Node *p=&n; p; p=p->next) ++p->data;
    
    return n;
}

Node n {1, new Node {2, nullptr}};
Node m = plusOne(n); // copy ctor, but what is "other" here? reference to what?
```

- temporary object created to hold result of plusOne
- other is a reference to temporary
  - copy ctor deep-copies the temp into m

BUT

- temporary is just going to be discarded as soon as statement

```cpp
Node m = plusOne(n); // is done
```

- wasteful to deep copy the temp
  - why not just steal the data instead?
    - shallow copy
    - saves cost of copy
  - need to be able to tell if other is a ref to a temp object or a standalone one

## R-value References

- r-value pure value
- l-value "location"

- Node && is a reference to a temporary object (r-value) of type Node
- write version of ctor that takes a Node &&

```cpp
struct Node {
    ...;
    Node (Node &&other); // move ctor
};

Node::Node(Node &&other): data{other.data}, next{other.next} {
    other.next = nullptr;
}

// Similarly
Node m;
m = addOne(n); // assignment from temporary
struct Node {
    ...;
    Node &operator=(Node &&other) {
        // move assignment operator, steal other's data, destroy old data
        // swap without copy
        ...;
       	using std::swap;
        swap(data, other,data);
        swap(next, other.next);
        
        return *this;
    }
}
```

# Move Assignment Operator

```cpp
struct Node {
    Node &operator(Node &&other) {
        using std::swap;
        swap(data, other.data);
        swap(next, other.next);
        
        return *this;
    }
};
```

# September 27

## Combine Copy/Move Assignment

```cpp
struct Node {
    ...;
    Node &operator=(Node other) { // unified assignment operator
        
    }
};
```

### Pass by Value

- invokes copy constructor if argument is l-value
- invokes move constructor (if there is one) if argument if r-value

#### NOTE:

- copy and swap can be expensive
  - hand coded operations may do less copying

## Consider

```cpp
struct Student {
	string name; // string is class
	Student(const string &name): name{name} {}
	// copies arg into the field (uses string's copy ctor)
    // what if name points to an rvalue
};


struct Student {
    string name;
    Student(std::string name): name{name} {} // Note copies twice
    // copies if lvalue
    // moves if rvalue
};

// Note name may come from rvalue, name ITSELF is an LVALUE


struct Student {
    string name;
    Student(string name): name{std::move(name)} {}
    // name{...} now a move construction
    // std::move forces value to be treated as an rvalue
}


struct Student {
    ...;
    Student(const Student &&other): // move ctor
    	name{other.name} {} // this is a copy construction
}


struct Student {
    ...;
    Student(Student &&other): // move ctor
    	name{std::move(other.name)} {}
    
    Student &operator=(Student other) { // unified assignment
        name = std::move(other.name);
        
        return *this;
    }
}
```

## Overview

- if no move operations defined, copy operations called
- if move operations defined, they will be called when the argument is a rvalue

# Copy / Move Elision

```cpp
Vector makeAVector() { return Vector{}; } // basic ctor

Vector v = makeAVector(); // move ctor? copy ctor?
```

- with g++: only ctor which prints anything is return Vector{}
- no call to copy or move ctor

### NOTE:

- in some cases, compiler is allowed, not does not have to, skip calling copy/move ctors
- in the example above
  - makeAVector writes its result directly into the space occupied by v in the caller, rather than copy it over

## ie

```cpp
Vector v = Vector {}; // Formally a basic construction and a copy constructor
// but compiler is REQUIRED to elide this copy construction, so basic ctor only
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   
Student bob { 90, 70, 80 };
Student bob = Student { 90, 70, 80 }
```

## ie

```cpp
void doSomething(Vector v) {...} // pass-by-value - copy/move ctor

doSomething(makeAVector());
// result of makeAVector written directy into the parameter
// no copy
```

- this is allowed even if dropping ctor calls would change the behavior of the program!!
  - e.g. if ctors print something

If we need all ctors to run:

```bash
g++14 -fno-elive-constructors ...
```

## Summary: Rule of 5 (Big 5)

If we need to customize any one of

- copy ctor
- copy assignment
- dtor
- move ctor
- move assignment

than usually you need to customize all 5

This is due to the fact that most circumstances which require customization are universal along the Big 5

## NOTE:

From now on, we assume Node & Vector have the Big 5 defined