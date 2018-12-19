# Problem 30: Logging

- want to encapsulate "logging" functionality and "add" it to any class

```cpp
template<typename T, typename Data> class Logger {
    public:
    void loggedSet(Data x) {
        cout << "Setting data to" << x << endl;
        static_cast<T*>(this)->set(x);
    }
    
    class Box: public Logger<Box, int> {
        friend class Logger;
        int x;
        void set(int y) {x=y;}
        
        public:
        Box(): x{0} {loggedSet(0);}
    };
}

Box b;
b.loggedSet(1);
b.loggedSet(4);
b.loggedSet(7);
```

- another approach?

```cpp
class Box {
    int x;
    
    public:
    Box(): x{0} {}
    void set(int y) {x=y;}
};

template<typename T, typename Data> class Logger: public T {
    public:
    void loggedSet(Data x) {
        cout << "Setting data to" << x << endl;
        set(x);
    }
};

using BoxLogger = Logger<Box, int>;
Boxlogger b;
b.loggedSet(1);
B.loggedSet(4);
b.loggedSet(7);
// no virtual method overhead
```

## Mixin Inheritance

- using inheritance to add features

### note:

- a special box is a subclass of box
  - then`Logger<Box>` is not a subclass of specialBox in the second solution
- On the other hand, for the CRTP solution
  - any subclass of Box has Logger as a superclass