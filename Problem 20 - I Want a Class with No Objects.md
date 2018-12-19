# Abstract Class

- once we create at least one virtual method
  - the class is a virtual class
- cannot be instantiated

```cpp
Student s; // illegal
Student *s = new Student; // illegal
```

## BUT

- can point to instances of concrete subclasses

```cpp
Student *s = new RegularStudent;
```

### Note:

- A class which is not __abstract__ is __concrete__

- Subclasses of abstract classes are also abstract unless they implement all pure virtual methods
- Abstract classes
  - used to organize concrete classes
  - can contain common fields, methods, default implementations