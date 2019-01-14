# A Big Unit on Object-Oriented Design



## System Modelling - UML (Unified Modelling Language)

- makes ideas easy to communicate, aids design discussion

A

```
Book // classname
-----
- title: String // fields (optional)
- author: String
# length: Integer
-----
+ getTitle(): String // methods (optional)
+ getAuthor(): String
+ getLength(): Integer
+ _isHeavy()_: Boolean
```

B->A

```
Text
-----
- topic: String
-----
+ getTopic(): String
+ isHeavy(): Boolean
```

C->A

```
Comic
-----
- hero: String
-----
+ getHero(): String
+ isHeavy(): Boolean
```

### Legend

```
- = private
# = protected
+ = public
italicized = virtual (pure or not) // not italics overwrote
italicized classname = abstract class
arrow = "is-a" relationship (inheritance)
diamond = "owns-a" relationship
```

- ''owns-a"

  - ```
    Car "owns-a" motor
    ```

  - _typically_ class composition (object fields)

  - ```cpp
    class Car {Motor m;};
    ```

### Note

- we distinguish between `int` and Integer `bool` and Boolean, etc
  - __NOT__ shorthand for C++
  - high level conceptual discussion

# Recall

![Alt text](https://g.gravizo.com/svg?
@startuml;
object Car;
object Motor;
Car *-- Saw;
@enduml;
)

- no independent existence for the motor
- copy / destroy car => copy / destroy motor

# November 6, 2018

![Alt text](https://g.gravizo.com/svg?
@startuml;
object Pond;
object Duck;
Pond o-- Duck;
@enduml;
)

- "has-a" relationship
- copy / destroy pond does not imply we destroy copy / destroy ducks
- typical implementation

```cpp
class Pond {vector<Duck*> ducks;};
```


## Ownership

- central to OO design in C++
- Every resource should be __owned__ by an object that releases it
- RAII
- raw pointer should __NOT__ be regarded as owning the memory it points to
- If you need ot point at the same object.
  - with several pointers one pointer should own it
    - be unique_ptr
    - rest should be raw
- Moving a unique pointer
  - transfer of ownership
- True shared ownership later

# Measures of Design Quality

## Coupling

- how strongly different modules depend on each other
  - low:
    - function calls with parameters / results of basic type
    - function calls with array / struct params
    - modules which affect each other's control flow
  - high:
    - modules access each other's implementation (friends)
- changes to one module affects other modules
  - harder to reuse individual modules

### ie

- function whatIsIt (`dynamic_cast`)
  - tightly coupled to the Book class hierarchy
  - must change the function if we create another Book subclass

## Cohesion

- how closely are elements of a module related to each other
- low: poorly organized code; hard to understand, maintain
  - arbitrary grouping (ie `<utility>`)
  - common, otherwise unrelated, maybe some common base code (ie `<algorithm>`)
  - elements manipulate state over the lifetime of an object (ie open/read/close files)
  - elements pass data to each other
- high:
  - elements cooperate to perform exactly one task

### Goal

__low__ coupling, __high__ cohesion

# SOLID Principles of OO Design

## Single Responsibility Principle

- a class should only have one reason to change
  - class should do one thing, not several
- any change to the problem specification requires a change to the program
- If changes to more than 2 different parts of the specification cause changes to the same class, SRP is violated

### Ex

- do not let your classes print things

```cpp
class ChessBoard {
    ...;
    cout << "Your move";
    ...;
};
```

- bad design
  - inhibits code reuse
- What if you want a version of your program that
  - communicates over different stream (files, networks)
  - works in another language?
  - uses graphics instead of text?
- Major changes to `ChessBoard` class, instead of reuse
  - violates SRP
  - must change this class if there is any change to the specification for
    - game rules
    - strategy
    - interface
    - etc
  - low cohesion
- Split these responsibilties up
  - one module (not main! as we cannot reuse it)
  - responsible for communication
  - pass info to the communications object and let it do the talking
- On the other hand..........
  - specifications that are unlikely to change may not need their own class
  - avoid needless complexity
- Judgement call

## Open-Closed Principle

classes, modules, functions, etc should be open to extension and closed for modification

- changes in program behavior should happen by writing __NEW__ code
  - extending functionality
  - __NOT__ changing old code

### Ex

![Alt text](https://g.gravizo.com/svg?
@startuml;
object Carpenter;
object HandSaw;
Carpenter *-- HandSaw;
@enduml;
)

- what if Carpenter buys a table Saw?
- this design is NOT open to extension, instead must change code

#### Solution

- Example of the Strategy Pattern
  - abstract over strategy

![Alt text](https://g.gravizo.com/svg?
@startuml;
object Carpenter;
object Saw;
object HandSaw;
object TableSaw;
Carpenter *-- Saw;
HandSaw --> Saw;
TableSaw ..> Saw;
@enduml;
)

## Also Note

```cpp
int countHeavy(const vector<Book*> &v) {
    int count = 0;
    for(auto &p : v) if(p->isHeavy()) ++count;
    return count;
}
```

- no changes needed if new Books invented
- us. whatISIt(`dynamic_casting`) not closed to modification

#### NOTE:

cannot really be 100% closed.

- some may require source modification
- Plan for the __MOST LIKELY__ changes
  - make your code closed with respect to those changes

# Recall

- SOLID OOP

# November 7, 2018

## Liskov Substitution Principle

- simply put
  - public inheritance must indicate an "is-a" relationship

But there is more to it:

- If B is a subtype of A, then we should be able to use an object b of type B in any context that requires an object of type A
  - __WITHOUT__ affecting the correctness of the program
- C++ inheritance rules already allow us to use the subclass in place of the super class object
- Informally:
  - a program should "not be able to tell", if it is using a superclass obj or a subclass obj
- More formally:
  - If an invariant I is true of class A, it must be true of class B
  - If an invariant is true of class `A::f`, it must hold for `B::f`
    - pre-condition, post-condition
      - If `A::f` has precondition $$p$$, and postcondition $$q$$
        - given $$p$$, calling `A::f` gives $$q$$ 
        - `B::f` must have precondition $$P'\impliedby P$$ and postcondition $$Q' \implies Q$$
  - If `A::f` and `B::f` behave differently, the difference in behavior must fall within what is allowed by the program's correctness specification

### Contravariance Problem

- arises any time you write a binary operator
  - ie a method with an "other" parameter of the same type as `*this`



![Alt text](https://g.gravizo.com/svg?
@startuml;
object Shape;
object Circle;
object Square;
Circle --|> Shape;
Square --|> Shape;
@enduml;
)

```cpp
class Shape {
    public:
    virtual bool operator==(const Shape &other) const;
};

class Circle: public Shape {
    public:
    bool operator==(const Circle &other) const override;
};

// DOES NOT COMPILE!
```

- violates Liskov substitution
  - A `Circle` is a `Shape`
  - A `Shape` can be compared with __ANY__ other `Shape`
  - Thus a `Circle` can be compared with any other `Shape`
  - C++ will flag this as a compile-time error

```cpp
virtual operator= // previous example
```

```cpp
class Circle: public Shape {
    public:
    bool operator==(const Shape &other) const override;
}
```

- In general, no satisfactory solution, but for this instance

```cpp
#include <typeinfo>

bool circle::operator==(const shape &other) const {
    if(typeid(other) != typeid(Circle)) return false;
    
    const Circle &other = static_cast<const Circle &>(other);
    // compare fields between *this, and other
}
```

### `dynamic_cast` VS `typeid`

```cpp
dynamic_cast<const Circle &>(other); // is other a Circle or a SUBCLASS of Circle?

typeid(other) == typeid(Circle); // is other precisely a Circle?

typeid(other); // returns an object of type type_info, garanteed to implement operator==
```

### Is a square a Rectangle?

```cpp
class Rectangle {
    int length; height;
    public:
    int getLength() const {...}
    int getHeight() const {...}
    virtual void setLength(int length) {...}
    virtual void setHeight(int height) {...}
    int area() const {return length*width;}
 };

class Square: public Rectangle {
    public:
    Square(int side): Rectangle{side, side} {}
    void setLength(int length) override {
        Rectangle::setLength(length);
        Rectangle::setWidth(length);
    }
    void setWidth(int width) override; // similar
};

int f(Rectangle &r) {
    r.setLength(10);
    r.setLength(20);
    return r.area(); // expect 200
}

Square s{1};
f(s); // returns 400
```

- `Rectangle`s have the property that length and width can vary independently
  - `Square`s do not
- This violates the Liskov Substitution Principle
- On the other hand
  - Immutable `Square` could substitute for Immutable `Rectangle`

![Alt text](https://g.gravizo.com/svg?
@startuml;
object RightAngleQuadrilateral;
object Rectangle;
object Square;
Rectangle --|> RightAngleQuadrilateral;
Square --|> RightAngleQuadrilateral;
@enduml;
)

- `RightAngleQuadrilateral`
  - no invariant on sides
  - `Rectangle`
    - can vary length and width independently
  - `Square`
    - length == width

# Recall

- subclass should conform with expected superclass behavior

# November 8, 2018

- can we constrain what subclasses can do?

## Consider

```cpp
class Turtle {
    public:
    virtual void draw() = 0;
};

class RedTurtle: public Turtle {
    public:
    void draw() override {
        drawHead();
        drawRedShell();
        drawTail();
    }
}

class GreenTurtle: public Turtle {
    public:
    void draw() override { // same sequence as GreenTurtle
        drawHead();
        drawGreenShell();
        drawTail();
    }
}
```

- code duplication
- also, how can we ensure that overrides always do these things?

```cpp
class Turtle {
    void drawHead();
    virtual void drawShell() = 0; // can we override private method? YES
    void drawTail();
    
    public:
    void draw() { // enforces order of rendering, only gives freedom to choose shell
        drawHead();
        drawShell();
        drawTail();
    }
};

class RedTurtle: public Turtle {
    void drawShell() override;
};

class GreenTurtle: public Turtle {
    void drawShell() override;
};
```

- subclasses only control drawing of shell
  - not steps
  - not head
  - not tail

## Template Method Pattern

- fill in the blanks method
- template for method in which constant are enforced and freedom given to less stuff

## NVI (Non-Virtual Interface) Idiom

- public virtual methods are simultaneously:
  - part of class's interface
    - pre / post conditions
    - class invariants
    - stated purpose
  - "hooks" for customization by subclasses
- somewhat contradictory
- All virtual methods should be private
  - all public methods should be non-virtual

```cpp
class DigitalMedia {
    public:
    virtual void play() = 0;
};

class DigitalMedia {
    public:
    void play() {
        doPlay(); // can add before/after code ie check Copyright?
    }
    
    private:
    virtual void doPlay() = 0;
}
```

- generalization of __Template Method Pattern__
  - puts every virtual function inside a template method

# Interface Segregation Principle

- many small interfaces is better than one large interface
- if a class has many functionalities, each client of the class should see only the functionality it needs

```cpp
class Enemy {
    public:
    virtual void draw(); // needed by UI
    virtual void strike(); // needed by game logic
};

class UI {
    vector<Enemy *> v;
};

class BattleField {
    vector<Enemy *> v;
};
```

- If we need to change the drawing interface, Battlefield must recompile for no reason
- same for UI
- unnecessary coupling between UI and BattleField

## One Solution - Multiple Inheritance

```cpp
class Draw {
    public:
    virtual void draw() = 0;
};

class Combat {
    public:
    virtual void strike = 0;
};

class Enemy: public Draw, public Combat {...};

class UI {
    vector<Draw *> v;
};

class Battlefield {
    vector<Combat *v> v;
};
```

- example of __Adapter Pattern__

## Adapter Pattern

- general use of adapter
  - class provides interface which is different from the desired one

```cpp
class NeededInterface {
    public:
    virtual void g();
}

class ProvidedClass {
    public:
    void f();
}

class Adapter: public NeededInterface, public/private ProvidedClass {
    // inheritance from ProvidedClass can be private
    // depends on if we want adapter to still support old interface
    void g() override {
        f();
    }
}
```

### Detour - Issue with Multiple Inheritance

![Alt text](https://g.gravizo.com/svg?
@startuml;
object A1;
object A2;
object B;
A1 : +a();
A2 : +a();
B --|> A1;
B --|> A2;
@enduml;
)

- compile error
- `B` has two `a()` methods

![Alt text](https://g.gravizo.com/svg?
@startuml;
object A;
object B;
object C;
object D;
A : +a();
B --|> A;
C --|> A;
D--|> B;
D--|> C;
@enduml;
)

- `D` has 2 `a()` methods and they are different

```cpp
class D: public B, public C {
    void f() {... B::a(), C::a();}
};

D d;
d.a(); // ambiguous
d.B::a();
d.C::a();
```

Or maybe there should be only one `A` base and therefore only one `a()`

- Deadly Diamond of Death

![Alt text](https://g.gravizo.com/svg?
@startuml;
object Book;
object Novel;
object Comic;
object GraphicNovel;
Book : -title;
Book : -author;
Book : -length;
Novel --|> Book;
Comic --|> Book;
GraphicNovel--|> Novel;
GraphicNovel--|> Comic;
@enduml;
)

```cpp
class B: virtual public A {...};
class C: virtual public A {...};
class D: public B, public C {...};

D.a(); // No longer ambiguous
```

- name resolution happens before private check so even if we make `a()` private in one of the classes, it still will not compile properly

#### _Eg_ `iostream` hierarchy

![Alt text](https://g.gravizo.com/svg?
@startuml;
object ios;
object istream;
object ifstream;
object istringstream;
object ostream;
object ofstream;
object ostringstream;
object iostream;
object fstream;
object stringstream;
istream--|> ios;
ostream--|> ios;
ifstream--|>istream;
istringstream--|>istream;
ofstream--|>ostream;
ostringstream--|>ostream;
iostream--|>istream;
iostream--|>ostream;
fstream--|>iostream;
stringstream--|>iostream;
@enduml;
)

### How will a class like `D` be laid out in memory?

#### Consider

| memory of C |
| ----------- |
| `vptr`      |
| A fields    |
| C fields    |

| memory of D                                         |
| --------------------------------------------------- |
| `vptr`                                              |
| A fields                                            |
| B fields                                            |
| C fields - Does not work, does not look like a `C*` |
| D fields                                            |

g++ has:

| memory of D |
| ----------- |
| `vptr`      |
| B fields    |
| `vptr`      |
| C fields    |
| D fields    |
| `vptr`      |
| A fields    |

# Recall

![Alt text](https://g.gravizo.com/svg?
@startuml;
object A;
object B;
object C;
object D;
A : +a();
B --|> A;
C --|> A;
D--|> B;
D--|> C;
@enduml;
)

| memory of C |
| ----------- |
| `vptr`      |
| A fields    |
| C fields    |

| memory of D                                         |
| --------------------------------------------------- |
| `vptr`                                              |
| A fields                                            |
| B fields                                            |
| C fields - Does not work, does not look like a `C*` |
| D fields                                            |

g++ apparently has:

| memory of D |
| ----------- |
| `vptr`      |
| `B` fields  |
| `vptr`      |
| `C` fields  |
| `D` fields  |
| `vptr`      |
| `A` fields  |

# November 13, 2018

| memory of D                      |
| -------------------------------- |
| `vptr` - ptr to `D`, ptr to `B`? |
| `B` fields                       |
| `vptr` - ptr to `C`              |
| `C` fields                       |
| `D` fields                       |
| `vptr` - ptr to `A`              |
| `A` fields                       |

`B b;`
| memory of B |
| ----------- |
| `vptr`      |
| `B` fields  |
| `vptr`      |
| `A` fields  |

`C c;`
| memory of C |
| ----------- |
| `vptr`      |
| `C` fields  |
| `vptr`      |
| `A` fields  |

## Note:

figuring where to find Super Class part requires a `vtable` check (run-time)

- B + C need to be laid out so that we can find the A part, but the distance is not known (depends on the run-time type of the obj)
  - Solution
    - location of the base object stored in `vtable`

## Also

- diagram does not simultaneously look like `A`, `B`, `C`, `D`
  - but slices of it do
- pointer assignment between 2 pointers is not guaranteed to preserve addresses

```cpp
D *d = ...;
A *a = d; // changes the address
```

- the above adds an offset to point to the A part

### Note

- `static_cast` and `dynamic_cast` under multiple inheritance will also adjust the value of the ptr
- `reinterpret_cast` will not!!

# Dependency Inversion Principle

- high-level modules whould not depend on low-level modules
  - both should depend on abstractions
- abstract classes should never depend on concrete classes

## Traditional Top-Down Design

```
high-level module -------uses----------> low-level module

word count -----uses------> keyboard reader
```

- what if I want to use a file reader
  - changes to details affect higher-level module

#### Dependency Inversion

```
high-level module -----uses----> low-level abstraction <|-------low-level module

word count ---uses---> interface <|------keyboard reader, file reader
```
![Alt text](https://g.gravizo.com/svg?
@startuml;
object high_level;
object interface;
object low_level1;
object low_level2;
high_level --> interface : uses;
low_level1 --|> interface;
low_level2 --|> interface;
@enduml;
)

- works over several layers as well

![Alt text](https://g.gravizo.com/svg?
@startuml;
object high_level;
object med_level;
object low_level;
high_level --> med_level : uses;
med_level --> low_level : uses;
@enduml;
)

![Alt text](https://g.gravizo.com/svg?
@startuml;
high_level --> med_interface : uses;
med_level --|> med_interface;
med_level --> low_interface : uses;
low_level --|> low_interface;
@enduml;
)

eg //
- when the timer hits some specified time, it rings the bell (calls `Bell::notify`, which rings the bell)
  ![Alt text](https://g.gravizo.com/svg?
  @startuml;
  timer o-- Bell;
  Bell : +notify();
  @enduml;
  )
- what if we want the timer to trigger other events?
- maybe more than 1?
  ![Alt text](https://g.gravizo.com/svg?
  @startuml;
  timer o-- Responder;
  Responder : +notify();
  Bell --|> Responder;
  Bell : +notify();
  Light--|> Responder;
  Light : +notify();
  @enduml;
  )
- dynamic set of responders
  ![Alt text](https://g.gravizo.com/svg?
  @startuml;
  Timer o-- Responder;
  Timer : +register(Responder);
  Timer : +unregister(Responder);
  Responder o-- Timer;
  Responder : +notify();
  Bell --|> Responder;
  Bell : +notify();
  Light--|> Responder;
  Light : +notify();
  @enduml;
  )
- but now `Responder` is depending on concrete Timer class
  - can apply DIP again
    ![Alt text](https://g.gravizo.com/svg?
    @startuml;
    Source o-- Responder;
    Source : +register(Responder);
    Source : +unregister(Responder);
    Timer --|> Source;
    Timer : +getTime();
    Responder : +notify();
    Bell --|> Responder;
    Bell o-- Timer;
    Bell : +notify();
    Light--|> Responder;
    Light o-- Timer;
    Light : +notify();
    @enduml;
    )
- If `Light`/`Bell`'s behavior depends on the time, they may need to depnd on the concrete timer for a `getTime` method
- could dependency invert this as well, if you want
- General Solution is _Observer Pattern_

## Observer Pattern
  ![Alt text](https://g.gravizo.com/svg?
  @startuml;
  Subject o-- Observer;
  Subject : +notifyObservers();
  Subject : +attach(Observer);
  Subject : +detach(Observer);
  Observer : +notify();
  ConcreteSubject --|> Subject;
  ConcreteSubject : +getState();
  ConcreteObserver o-- ConcreteSubject;
  ConcreteObserver --|> Observer;
  ConcreteObserver : +notify();
  @enduml;
  )

Sequence of Calls:

1. Subject's state changes
2. `Subject::notifyObservers` calls each observer's notify
3. Each `Observer` calls `ConcreteSubject::getState` to query the state and react accordingly

# More Design Patterns

## Factory Method Pattern

- when you do not know exactly what kind of object you want, and your preferences may vary
- also called the __Virtual Constructor Pattern__

  - Constructors can __never__ be `virtual`
- suppose we create a game, need sequence of enemies, exact type of enemies not known
  - type of constructor varies
  - randomly generated
    - more turtles in easy levels
    - more bullets in harder levels
  - __Strategy Pattern__ applied to objet creation

  ![Alt text](https://g.gravizo.com/svg?
  @startuml;
  turtle --|> enemy;
  bullet --|> enemy;
  @enduml;
  )

  ![Alt text](https://g.gravizo.com/svg?
  @startuml;
  easy --|> level;
  hard --|> level;
  @enduml;
  )

```cpp
class Level {
	public:
    virtual Enemy *getEnemy() = 0;
};

class Easy: public Level {
    public:
    Enemy *getEnemy() override {
        // mostly turtles
    }
};

class Hard: public Level {
    public:
    Enemy *getEnemy() override {
        // mostly bullets
    }
};

Level *l = ...;
Enemy *e = l->getEnemy();
...;
```

- should apply NVI and `unique_ptr`

## Decorator Pattern

- add / remove functionality to / from objects dynamically

eg // 

- add menu / scrollbar to basic windows

  ![Alt text](https://g.gravizo.com/svg?
  @startuml;
  Component : +operation();
  ConcreteComponent --|> Component;
  ConcreteComponent : +operator();
  Decorator --|> Component;
  Decorator o-- Component;
  ConcreteDecoratorA --|> Decorator;
  ConcreteDecoratorA : +operation();
  ConcreteDecoratorB --|> Decorator;
  ConcreteDecoratorB : +operation();
  @enduml;
  )

  - Every Decorator __"IS-a"__ `Component` and __"HAS-a"__ `Component`
  - `WindowWithScrollBar` IS a window and has a ptr to the underlying plain window
  - `WindowWithMenu` is a `Window ` and has a ptr to `WindowWithScrollbar`, which has a ptr to a `Window`

  ```cpp
  WindowInterface *w = new WindowWithMenu {
      new WindowWithScrollBar {
          new Window {};
      }
  }
  ```

  - Pizza Example in repository

## Visitor Pattern

- for implementing __double dispatch__
- method chosen based on the runtime type of __2__ objects, rather than just 1

![Alt text](https://g.gravizo.com/svg?
  @startuml;
  Turtle --|> Enemy;
  Bullet --|> Enemy;
  @enduml;
)

![Alt text](https://g.gravizo.com/svg?
  @startuml;
  Stick --|> Weapon;
  Rock --|> Weapon;
  @enduml;
)

- what is the effect of striking an enemy with a weapon depends on __both__ the enemy __and__ the weapon
- C++ 
  - `virtual` methods are dispatched on the type of the __receiver__ object, and __not__ method parameters
  - no way to specify 2 receiver objects
- visitor pattern
  - combine overriding with overloading to do a 2-stage dispatch

```cpp
class Enemy {
    public:
    virtual void beStruckBy(weapon &w) = 0;
};
```

# MVC Architecture

![Alt text](https://g.gravizo.com/svg?
@startuml;
Model --|> View : update;
Controller --|> Model : parse;
User--|> Controller : input;
View --|> User : sees;
@enduml;
)



## Model-View

- observer pattern
  - command line output
  - graphical output

## Model

![Alt text](https://g.gravizo.com/svg?
@startuml;
Maze --|> Model;
Model *-- Controller;
Keyboard --|> Controller;
CursesKeyboard --|> Controller;
Model *--View;
Standard --|> View;
Curses --|> View;
Written --|> View;
Written o-- Maze;
@enduml;
)

- look at __Command Pattern__
- 



## View

## Controller

## ncurses

# Recall

## Visitor Pattern

- depends on caller and callee

# November 15, 2018

```cpp
class Enemy {
    virtual void beStruckBy(Weapon &w) = 0;
};

class Turtle: public Enemy {
    void beStruckBy(Weapon &w) override {w.strike(*this);} // overload
};

class Bullet: public Enemy {
    void beStruckBy(Weapon &w) override {w.strike(*this);}
};

class Weapon {
    public:
    virtual void strike(Turtle &t) = 0; // override
    virtual void strike(Bullet &b) = 0;
};

class Stick: Weapon {
    public:
    virtual void strike(Turtle &t) override {/* smt */} // override
    virtual void strike(Bullet &b) override {/* smt */}
};

// etc

Enemy *e = new Bullet {...};
Weapon *w = new Rock {...};
e->beStruckBy(*w); // what happens?
```

- `Bullet::beStruckBy` runs (virtual method dispatch)
  - calls `Weapon::strike(Bullet &)` where `*this` is Bullet
    - known at compile-time
    - overload resolution
  - virtual method resolves to `Rock::strike`(Bullet &)
- Coupling
  - but more structured solution than `dynamic_cast`
- Visitor Pattern can also be used to add functionality to a class hierarchy without adding new virtual methods
  - Add a visitor to the `Book` Hierarchy

```cpp
class Book {
    public: virtual void accept(BookVisitor &v) {v.visit(*this);}
}; //etc

class BookVisitor {
    public:
    virtual void visit(Book &b) = 0;
    virtual void visit(Text &b) = 0;
    virtual void visit(Comic &b) = 0;
};
```

- for example, categorize and count

  - For Books by author
    - Texts
    - Comics
    - Topic
    - Hero
  - add virtual method to the Book hierarchy
  - __OR__ write a visitor

  ```cpp
  class Catalogue: public BookVisitor {
      public:
      map<string, int> theCat;
      void visit(Book &b) override {++theCat[b.getAuthor()];}
      void visit(Text &t) override {++theCat[b.getTopic()];}
      void visit(Comic &c) override {++theCat[b.getHero()];}
  }
  ```

## Note

- this will not compile!
  - circular include dependencies
  - `book.h`, `BookVisitor.h`, include each other
    - include guard prevents multiple inclusion
    - whichever ends up occurring first will refer to things no yet defined
- Needless `#include` create artificial compilation dependencies and slow down compilation
  - if we include something that is not needed
    - causes compilation dependencies in `Makefile`
      - changes in the thing we include propagate outwards
      - or prevent compilation all together

```cpp
class A {...} // A.h

class B {
    A a;
};

class C {
    A *a;
};

class D: public A {
    ...;
};

class E {
    A f(A);
};

class F {
    A f(A a) {a.someMethod();}
};

class G {
    t<A> x; // where t is a template
};
```

- needs include:
  - B
    - size
  - D
    - size
  - F
    - calls `a`'s method
- can forward declare
  - C
    - just pointer
  - F
    - size does not change based on A
    - header file not dependent on A
- special case:
  - G
    - depends on how G uses A
      - should collapse to one of the other cases
- Note that class F only needs an include because method `f`'s implementation is preset and uses a method of A
  - a good reason to keep implementation in `.cc` file
- Where possible:
  - forward declare

```cpp
#include <iosfwd>
```

```cpp
class A {}; class A2 {}; class A3 {};

class B {
    A1 a1;
    A2 a2;
    A3 a3;
};
// compilation dependency

// b.h
class BImpl;
class B {
    unique_ptr<BImpl> mImpl; // ptr to implementation  
};

//bimpl.h
struct Bimpl {
    A1 a1;
    A2 a2;
    A3 a3;
};
```

## pImpl Idiom

- breaks compilation dependency
- ptr have non-throwing swap
  - can provide the string guarantee on a B method
    - copying the impl into a new Bimpl structure (heap-allocated)
      - method modifies the copy
    - if anything throws, discard the new structure (easy and automatic with `unique_ptrs`)
    - if all succeeds, swap the impl struct ptrs (no throw)
    - previous impl is destroyed automatically by smart pointer

```cpp
class B {
    unique_ptr<BImpl> pImpl;
    ...;
    void f() {
        auto temp = make_unique<BImpl> (*pImpl);
        temp->doSomething();
        temp->doSomethingElse();
        std::swap(pImpl, temp); // nothrow
    } // strong guarantee
};
```

# Return to Abstraction