# Problem 16 - Insert / Remove in the middle

 method like `vector<T>::insert(size_t i, const T &x)` is easy

same method for `list<T>` would be slow (linear traversal)

- use an iterator, can be faster

```cpp
template<typename T> class Vector {
    public:
    iterator insert(iterator posn, const T &x) {
        increaseCap();
    }
}
```

- add new iterm
- shuffle iterator down (`move_if_no_except`)
- return iterator to point of insertion

## Is insert Exception safe?

- assuming T's copy and move operations are exception safe (at least basic guarantee)
  - may get partially shuffled vector
  - but will still be a valid vector!

## Note

If you have other iterators pointing at the vector

- iterators are no longer guaranteed to point at same items `:(`

### Convention

after a call to `insert` or `erase`, all iterators pointing after the point of insertion or erasure are considered invalid and should NOT be used

### Similar

if a reallocation happens

- all iterators are invalidated

#### Exercise

- write `erase` 
  - remove item pointed to by iterator
  - return iterator to the point of erasure
- write `emplace`
  - like insert, but takes ctor arguments

## BUT

- we have a problem with push_back
  - If placement new throws
    - vector stay the same
    - iterators were  invalidated

### Fix

- allocate new array
- place the new item
- copy or move old items
- delete

# Recall

```cpp
void pop_back();
T pop_back(); // would call copy ctor on return, possibly move (assume copy), then dtor on vector item
// if copy ctor throws
// dtor call will not happen
// not exception safes
```