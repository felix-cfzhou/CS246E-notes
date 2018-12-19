# Problem 33 - A fixed-size allocator

Let us write one

- expect: faster than the built-in
  - no need to keep track of sizes
- many traditional allocators
  - give pointers 1 word in
  - extra overhead to store size
- saves space
  - no need for hidden size field
- saves time
  - not need to hunt for block of right size

## Overview

1. create pool of memory

   - array large enough to hold `n` `T` objects

2. when a slot in the array is given to the client, it will act as a `T` object

   - when we have it
     - act as node in a linked list
   - when client has it, as `T` object
   - store an `int` in each slot
     - index (not ptr) of next free slot
   - store index of first free slot

   ## Allocation

```
1 | 2 | 3 | 4 ....... | -1
store index fo the next available slot

allocate from the front
allocated | 2 | 3 | 4 | ... | -1 || store 1 as first free
allocated | allocated | 3 | 4 | ... | -1 || store 2 as first free
```

## Deallocation

```
add to front of list

free the first one: 
2 | allocated | 3 | 4 | ... | -1 || store 0

free 2nd:
2 | 0 | 3 | 4 | ... | -1 || store 1
```

- both alloc / dealloc are constant time and VERY fast

## Implementation

### Unions

- can have ctors and pick one of two fields

## allocator

```cpp
template<typename T, int n> class FixedSizeAllocator {
    union Slot { // large enough to hold max(T, int), usable as an int
        int next;
        T data;
        
        Slot(): next{0} {}
    };
    
    Slot theSlots[n];
    int firstAvailable = 0;
    
    public:
    FixedSizeAllocator() {
        for(int i=0; i<n-1; ++i) theSlots[i].next=i+1;
        theSlots[n-1] = -1;
    }
    
    T *allocate() noexcept {
        if(firstAvailable == -1) return nullptr;
        
        T *result = &(theSlots[firstAvailable].data);
        firstAvailable = theSlots[firsAvailable].next;
        
        return result;
    }
    
    void deallocate(void *item) noexcept {
        int index = (static_cast<char *>(item) - reinterpret_cast<char *>(theSlots)) / sizeof(Slot);
        theSlots[index].next = firstAvailable;
        firstAvailable = index;
    }
};
```

## student

```cpp
class Student final {
    int assns, mt, final;
    static FixedSizeAllocator<Student, SIZE> pool; // how many slots do we want?
    
    public:
    Student(...): ...;
    static void *operator new(size_t size) {
        if(size != sizeof(Student)) throw std::bad_alloc {};
        
        while(true) {
            void *p = pool.allocate();
            if(p) return p;
            auto h = std::get_new_handler();
            if(h) h();
            else throw std::bad_alloc {};
        }
    }
    
    static void operator delete(void *p) noexcept {
        if(!p) return;
        pool.deallocate(p);
    }
};

fixedSizeAllocator<Student, SIZE> Student::pool;
```



## main.cc

```cpp
int main() {
    Student *s1 = new Student; // custom allocator
    Student *s2 = new Student;
    delete s1; // custom deallocator
    delete s2;
}
```

### Where does the memory for s1, s2 reside?

#### A

- static memory
- NOT the heap
- depending on implementation
  - pool can be
    - static memory
    - stack
    - heap

## Notes:

- use a union to treat a slot as both an `int` and a `T` object
- ensures no memory is wasted on book keeping
- could have implemented as struct
  - have both fields
  - alternative:
    - store "next" index adjacent to the `T` object

```
[next| T][next| T] .....
```

- union has advantage of no waste memory
- disadvantage:
  - if access a dangling `T` pointer
    - corrupt linked list by overwriting
    - heap is GONE

```cpp 
Student *s = new Student;
delete s;
s->stAssns(...); // probably corrupts the list :(
```

## Lesson: following dangling pointers can be very dangerous

- with a struct, the next field is before the pointer usually
  - harder to corrupt :)

```cpp
reinterpret_cast<int *>(s)[-1] = ...; // no accident
```

## Unions

- if one field has a ctor
  - union needs a ctor
    - as it is a union, ctor should only initialize 1 field

### On the other hand

- if we have a struct
  - difficulty if `T` has no default ctor

```cpp
struct Slot {
    int next;
    T data;
};

Slot theSlots[n]; // cannot do this if T has no defined ctor
// cannot do operator new or placement new as we are writing operator new
// creat char array and cast as etc etc
```

## Also

- use indexes vs ptrs?
- `int` smaller than ptrs
- So we waste no memory as long as `T` is at least as large as an `int`
- do waste memory if `T` is smaller than an `int`
- could use smaller type for index
  - `short`
  - `char`
- as long as less items than max index size
  - make index type a parameter of the allocator template

## Why is `class Student` `final`?

- fixed size allocator
- subclass may be larger
  - will not work with out allocator
  - but would still use it
- options
  1. make the class final
  2. check the size, throw if the size is not right
  3. check the size, use global operator new/delete if size if not right
  4. give derived class own allocator