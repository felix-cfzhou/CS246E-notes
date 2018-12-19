# CS 246E

- DC 3110 bmlusma@
- tutorial MANDATORY

# September 6

Read 2.2, 4.3

## About the Course
- Not about C++
- ABSTRACTION

## About C++
- brings abstraction and type safety from C

### Abstraction
- shield users from shit
- meant to be easier than C

#### Strings
- buffer overview
- null terminator

#### memory
- malloc, etc

#### C++
- built in string type
  - new, delete
- vector

### C++ is easy?
- write robust libraries which is applicable to wide variety of uses
- designed abstractions to fit many scenarios as possible

### C++ is hard
- construction of abstractions

## C++ is easy because it is hard :)

## Linux - must use!
- putty
  - linux.student.cs.uwaterloo.ca
  - enable X11 forwarding
- Mac / Linux - terminal
  - ssh -Y $userid@linux.student.cs.uwaterloo.ca
  - install XWindows server
    - XMing (Win), XQuartz (Mac)
      - xclock, xeyes

## Problem 1 - Program IO

Running Program from command line:

```bash
./program-name
path/to/program-name
```

Input

```bash
./program-name arg1 arg2 ... argn
# OS writes args into program memory
```
1. example
    - Code
    - n+1 (argc)
    - ... -> ./my-program
    - ... -> arg1
    - ...
    - ... -> argn (...argv)
    - Heap
    - Stack
2. another one
  - ./program-name (then type shit)
  - input comes through standard input stream (stdin) instead
  - stdin -> program -|-> stderr _> stdout
    - stderr -> error messages
    - stdout -> screen by default
    - buffering
      - printing to screen one of most expensive
      - pay per char or pay per batch
    - stout maybe buffered
    - things can be connected to other sources/destinations, e.g. files

Redirection

```bash
./my-program < infile > outfile 2> errfile
```

### C:

```c++
#include <stdio.h>

void echo(FILE *f) {
    for(int c=fgetc(f); c!=EOF; c=fgetc(f))s putchar(c);
    // fgetc fetch from file
    // putchar put to stdout
}

int main(int argc, char* argv[]) {
    // argc # of command line args
    // argv[0] is program name
    // ...
    // argv[argc] == NULL
    if(argc == 1) {
        echo(stdin);
    }
    else {
        for(int i=1; i<argc; ++i) {
            FILE *f = fopen(argv[i], "r");
            echo(f);
            fclose(f);
        }
    }
    
    return 0;
    // convention - 0 is OK
    // echo $? - status code of most recently run program
}
```

#### Observe: command-line args / input from stdin - 2 different programming techniques

##### compile:

```bash
gcc -std=c99 -Wall myprogram.c -o myprogram
```

#### Simplification of Linux 'cat' command

```bash
cat file1 file2 file3
```
- opens files sequentially and prints contents one after the other
```bash
cat # -echoes stdin
cat < file1
# file1 used as a source for stdin
# shell (not cat) opens the file
# displays file1
```

### C++

write cat program in C++?

- program above valid in C++

#### The "C++" way

- command-line args, same as C

```c++
#include <iostream>

int main() {
    int x, y;
    std:cin >> x >> y;
    std:cout << x+y << std::endl;
}

// std::cin : std::istream
// std::cout : std::ostream
// std::cerr : std::ostream
// >> input operator cin >> c - populates x as a side-effect, returns cin
// << ouput operator cout << x+y - prints x+y as a side-effect, returns cout
// return allows chaining
```

##### File access:

- programming language indifferent
- open file
- close file

```c++
#include <fstream>

std::ifstream f{"name-of-file"}; // c++ initialization syntax, implicitly opens file
char c;
while(f>>c) {
    // f>>c both reads into var c and evaluates to f
    // streams in C++ can be implicity convert to bool
    // true - all good, false - failed
    std::cout << c;
}

// file close called automatically when var f is out of scope
```

# September 11, 2018

- stream input skips whitespace

to include whitespace:

```cpp
std::ifstream f {'name-of-file'};
f >> std::noskipws;
// io manipulator
// changes how stream is handled
char c;
while(f >> c) {
    std::cout << c;
}
```

## Note (important):

- not explicit calls to fopen / fclose
- initialization f opens the file
- when f's scope ends, fie is closed.

## Cat in C++

```cpp
# include <iostream>
# include <fstream>

using namespace std; // Avoids having to say std::

void echo (istream f) {
    char c;
    f >> noskipws;
    while(f >> c) cout << c;
}

int main(int argc, char *argv[]) {
    if(argc == 1) echo (cin);
    else for (int i=1; i<argc; ++i) {
        ifstream f {argv[i]};
        echo(f);
    }
}

/*
Does not work, or even compile
cin - istream - echo takes an istream
f - ifstream - is echo(f) a type mismatch
Nope :) - ifstream is a subtype of istream
Any ifstream can be treated as istream
foundational concept in OOP
*/

/*
issues lies in echo func
- cannot write func that takes an istream the way echo does
- reason? - important cpp discussion
*/
```

Compare

```cpp
int x;
scanf("%d", &x);

// and

int x;
cin >> x;
```

C and C++ are pass-by-value languages.

scanf needs the address of x in order to change x, via pointers.

Why not cin >> &x?

C++ has another ptr-like type:

## References

```cpp
int y = 10;
int &z = y;
// z is an l-value reference to y
// similar to int *const z = &y
// auto-dereferences

z=12;
// NOT *z=12;
// y == 12
// z acts as y
// alias of y
```

### Rules

- l-value references must be initialized, and must be initialized to something which has an address

```cpp
int &z = y; // OK
int &x = 4; // NOPE
int &x = a+b; // NOPE

// NO pointer to a ref
int &*x; // NOPE

// Reference to pointer is OK
int *&x = ...; // OK

// Reference to a reference not OK
int &&r = z; // NOPE

// Array to reference not OK
int &z[3]= {...} // NOPE
```

### Can

- pass as func param

```cpp
void inc(int &n) {
    n+=1; // no ptr deference
}

int x=5;
inc(x);
cout << x; // now 6
```
#### Default choice

```cpp
cin >> x; // works as x is passed by reference

istream &operator >> (istream &in, int &x);

// now consider struct ReallyBig {...};
int f(ReallyBig rb) {...}
// no need to say struct ReallyBig in C++
// pass-by-value, causes copy to stack, slow

int g(ReallyBig &rb) {...}
// reference - no copy being done - fast
// but - possibly propagates changes to the caller
// Can we get quick pass into func and garantee struct stays the same

int h(const ReallyBig &rb) {...}
// fast - no copy - h cannot change rb

// Prefer pass-by-const-ref over pass-by-value for anything larger than a ptr
// UNLESS the func needs to make a copy anyways, then use pass-by-value

// ALSO
int f(int &n) {...}
int g(const int &n) {...}

f(5); // fails to compile as we cannot initialize l-value ref to a literal value
g(5);
// perfectly legal :), since n can never be changed, compiler allows this
// 5 stored in temp location, so n points at smt
```

## Back to cat

```cpp
void echo(istream f) {...}
// f passed by value
// copying streams is not allowed

void echo(istream &f) {...}
// everything else constant
// OK
```

### To compile

```bash
g++ -std=c++14 -Wall myprogram.cc -o myprogram
./myprogram
```

## Separate Compilation

- put echo in its own module:

```cpp
// echo.h:
void echo(istream &f);

// echo.cc:
# include "echo.h"
void echo(istream &f) {...}

// main.cc:
# include <iostream>
# include <fstream>
# include "echo.h"

int main(...) {
    ...echo(cin);
    ...;
    ...ifstream f{argv[i]};
    ...;
    ...echo(f);
    ...;
}
```

### actual compilation command

```bash
g++14 echo.cc
# compiles BUT
# does not LINK as no main

g++14 main.cc
# link error as well
# no echo definition


# Seperate Compilation
g++14 -c echo.cc # ouput echo.o
g++14 -c main.cc # output main.o
g++14 echo.o main.o -o myprogram # linker
```

### Advantages

only have to recompile the parts we change and relink

- Note that changing echo.h means we need to recompile everything and link

# September 12, 2018

## Linux Commands

### Common Commands

```bash
ls
pwd
cat
rm
cp
mv
cd
```

### Less Common Shit

```bash
touch file # updates timestamp or creates file
mktemp # makes temp file, returns location of file ie /tmp/tmp4u19rhwief98
mkdir
diff
echo
wc
less file # scrolling view of file
man
head -10 /usr/share/dict/words
tail -10 /usr/share/dict/words
tr # translate? 
```

```bash
# arg: -la
ls $(cat arg) # => ls -la
```

## C++

```cpp
#include <cstdlib>
#include <sys/wait.h>

int sys(std::string s) {
    int res = system(s.c.str()); // converts to system string
    
    return WEXITSTATUS(res);
}

bool samefile(std::string file1, std::string file2) {
    return sys("diff " + file1 + " " + file2) == 0;
}

int main(int argc, char* argv[]) {
    std::string filename = argv[1];
    int copies = 0;
    
    FILE *f = popen("ls", "r");
	char buf[128];

	while(fscanf("%s", buf) != EOF) {
    	std::string newFile = buf;
    	if(samefile(filename, newFile)) ++copies;
	}
    
    pclose(f);
    
    for(char c='1', c<='0' + 10 - copies, ++c) {
        sys("cp " + filename + " copy" + c);
    }
}
```

# September 20, 2018

## Advantages of Constructors

- automatically called
- default parameters

```cpp
struct Student {
    ...;
    Student(int assns=0, int mt=0, int final=0) {
        this->assns = assns;
        ...;
    }
}

Student laura {70}; // 70 0 0
Student newKid; // 0 0 0
```

- overloading
- sanity checks

### Note:

Every class comes with a built-in default constructor

```cpp
Node n; // calls default constructor
```

- default constructs all fields that are objects
  - goes away if constructor is provided

```cpp
struct Node {
    int data;
    Node *next;
    
    Node (int data, Node *next=nullptr) {
        this->data = data;
    }
}

Node n {3}; // OK
Node n; // NOPE
```

## Object Creation Protocol

when an object is created:

1. space is allocated
2. (later)
3. Fields constructed in declaration order
   - (field constructors are called for fields that are objects)
4. constructors body runs

Field initialization should happen in step 3, but constructor body is step 4

Consequence: object fields could be initialized twice:

```cpp
#include <string>
struct Student {
    string name; // object
    
    Student(const string &name) {
        this->name = name;
    }
}

Student mike {"mike"};
// name default-initialized to empty string
// reassigned in step 4
```

### To fix: Member Initialization List (MIL)

```cpp
struct Student {
    ...;
    Student(const string &name, int assns, int mt, int final): name{name}, assns{assns}, mt{mt}, final{final} {}
    // outside brackets must be field names
    // inside braces follows normal scoping rules
}
```

#### Note:

- MIL must be used for fields that are
  - constants
  - references
  - objects w/o default constructors

```cpp
Student(...): name(name) ... {};
Student mike("mike");

int x(5);
```

- MIL SHOULD be used as much as possible

### Careful: single-argument constructors

```cpp
struct Node {
    ...;
    Node(int data, Node *n=nullptr): data{data}, next{next} {}
};
// single argument constructors create implicit conversions
Node n {4}; // OK
Node n = 4; // OK - int implicitly converted to Node.
```

```cpp
void f(Node n);
f(4); // OK - maybe trouble?

f(x); // YIKES
```

```cpp
struct Node {
   	...;
    explicit Node(...): ... {}
    // disables the implicit conversion
};
```

## Object Destruction

- method called the destructor (dtor) which runs automatically.
- build-in destructor
  - any object fields have their destructors called

### Object Destruction Protocol

1. destructor body runs
2. fields destructed (destructors are called on fields that are objects) in reverse declaration order
3. (later)
4. space deallocated

```cpp
struct Node {
    int data;
    Node *next;
    // no objects, nothing is done
}

Node *n = new Node {3, new Node {4, new Node {5, nullptr}}};
delete n;
```

### Writing our own destructor

```cpp
struct Node {
    ...;
    
    ~Node() {
        delete next;
    }
}

delete n; // frees whole list
```

Also:

```cpp
{
    Node n {1, new Node {2, new Node {3, nullptr}}}; // first is stack allocated
    ...;
    
} // scope of n ends; n destructor will run; whole list is freed
```

## Objects

- constructor always runs when created
- destructor always runs when destroyed

## Back to Vector Problem

### vector.h

```cpp
#ifndef
#def VECTOR_H
namespace CS246E {
    struct Vector {
        size_t size, cap;
        int *theVector;
        
        Vector();
        size_t size();
        int &itemAt(size_t i);
        void push_back(int n);
        void pop_back();
        ~Vector();
    };
}
#endif
```

### vector.cc

```cpp
namespace {
    void increaseCap (const Vector &v) {...}
}

CS246E::Vector::Vector(): size{0}, cap{0}, theVector{new int [cap]} {}
	
```

# September 13, 2018

## Linux Make

- create a file called "MakeFile"

```makefile
my-program: main.o echo.o
	g++ main.o echo.o -o my-program
main.o: main.cc echo.h
	g++ -std=c++14 -Wall -c main.cc
echo.o: echo.cc echo.h
	g++ -std=c++14 -Wall -c echo.cc
.PHONY: clean
clean:
	rm my-program *.o

# No aliases allowed
# NOTE: this fucking must be a tab
# make clean removes all binaries
```

### How?

- list a directory in long form

```bash
ls -l
# - type
# rw-r----- owner/group/other permissions
# 1 # links
# j2smith owner
# j2smith group
# 25 size
# Sep 9 15:27 last modified date/time
# echo.cc name
```

- based on modified time
- starting at the leaves of the dependency graph
- if the dependency is newer than the target, rebuild the target ... and so on

ie

1. echo.cc is newer than echo.o - echo.o out of date, rebuild
2. echo.o is now newer than my-program - rebuild my-program

Shortcuts - user vars

```makefile
CXX = g++ # which compiler
CXXFLAGS = -std=c++14 -Wall -g # compiler flags
EXEC = my-program # name of program
OBJECTS = main.o echo.o # name of objects

${EXEC}: ${OBJECTS}
	${CXX} ${OBJECTS} -o ${EXEC}
main.o: main.cc echo.h # omit recipes, make "guesses" the right ones
echo.o: echo.cc echo.h # omit recipes, make "guesses" the right ones
.PHONY: clean
clean:
	rm ${OBJECTS} ${EXEC}
```

v3

```makefile
CXX = g++ # which compiler
CXXFLAGS = -std=c++14 -Wall -g -MMD # compiler flags
EXEC = my-program # name of program
OBJECTS = main.o echo.o # name of objects
DEPENDS = ${OBJECTS: .o=.d}

${EXEC}: ${OBJECTS}
	${CXX} ${OBJECTS} -o ${EXEC}
- include ${DEPENDS}
.PHONY: clean # specifies that clean is NOT a file
clean:
	rm ${OBJECTS} ${DEPENDS} ${EXEC}
```

## Always Use MakeFiles!

- create MakeFile before start coding