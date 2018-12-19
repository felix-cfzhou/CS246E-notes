# Problem 28 - Resolving Method Overrides At Compile-Time

## Recall: Template Method Pattern

```cpp
class Turtle {
    public:
    void draw() {
        draw Head();
        draw Shell();
        draw Tail();
    }
    
    private:
    void drawHead();
    virtual void drawShell() = 0; // vtable lookup
    void drawTail();
};

class RedTurtle: public Turtle {
    void drawShell() override;
};
```

- Consider:

```cpp
template<typename T> class Turtle {
    public:
    void draw() {
        drawHead();
        static_cast<T *> (this)->drawShell();
        drawTail();
        
    }
    
    private:
    void drawHead();
    void drawTail();
};

class RedTurtle: public Turtle<RedTurtle> {
    friend class Turtle;
    void drawShell();
};

class GreenTurtle: public Turtle<GreenTurtle> {...};
```

- no virtual methods

- no vtable lookup

- Drawback

  - no relationship between RedTurtle and GreenTurtle
  - cannot store a mix of them in one container
  - give Turtle a parent

```cpp
template<typename T> class Turtle: public Enemy {...};
```

- can store `RedTurtles` with `GreenTurtles`
  - cannot access draw method
  - give Enemy a virtual draw?

```cpp
class Enemy {
    public:
    virtual void draw() = 0;
};
```

- but there _WILL_ be vtable lookup
- But we may be reducing several would be virtual helper calls to one
  - still a W