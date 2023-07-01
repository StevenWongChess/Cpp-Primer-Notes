### Ch 9. Sequential Containers

The container classes share a common interface, which each of the containers extends in its own way. This common interface makes the library easier to learn; what we learn about one kind of container applies to another

#### 9.1 Overview of the Sequential Containers

6 sequential contains => see page. 326 Table 9.1

- all provide fast sequential access to their elements
- Trade-off between
  - The costs to add or delete elements to the container
  - The costs to perform nonsequential access to elements of the container

 For reasons we’ll explain in § 13.6 (p. 531), the new library containers are dramatically faster than in previous releases

- Modern C++ programs should use the library containers rather than more primitive structures like arrays

Ordinarily, it is best to use vector unless there is a good reason to prefer another container



#### 9.2 Container Library Overview

The operations on the container types form a kind of hierarchy:

- Some operations (Table 9.2 p. 330) are provided by all container types

- Other operations are specific to the sequential (Table 9.3 p. 335), the associative Table 11.7 p. 438), or the unordered (Table 11.8 p. 445) containers

- Still others are common to only a smaller subset of the containers

In general, each container is defined in a header file with the same name as the type

Need to provide `constructor` if there is no default one

```c++
// assume noDefault is a type without a default constructor
vector<noDefault> v1(10, init); // ok: element initializer supplied 
vector<noDefault> v2(10); // error: must supply an element initializer
```

###### 9.2.1 Iterators

With 1 exception, the container iterators support all the operations listed in Table 3.6 (p. 107)

- the `forward_list` iterators do not support the `--`

The iterator arithmetic operations listed in Table 3.7 (p. 111) apply ONLY to iterators for `string`, `vector`, `deque`, and `array`

###### 9.2.2 Container Type Members

`reverse iterator` is an iterator that goes backward through a container and inverts the meaning of the iterator operation

- more to say in § 10.4.3 (p. 407)

Element-related type aliases are most useful in generic programs, which we’ll cover in Chapter 16

###### 9.2.3. `begin` and `end` Members

4 kinds of iterators

```c++
list<string> a = {"Milton", "Shakespeare", "Austen"};
auto it1 = a.begin(); // list<string>::iterator
auto it2 = a.rbegin(); // list<string>::reverse_iterator
auto it3 = a.cbegin(); // list<string>::const_iterator
auto it4 = a.crbegin();// list<string>::const_reverse_iterator
```

###### 9.2.4 Defining and Initializing a Container

How to? => see page. 335 Table 9.3

Every container type defines a default constructor (§ 7.1.4, p. 263). 

- With the exception of array, the default constructor creates an empty container of the specified type
- Again excepting array, the other constructors take arguments that specify the size of the container and initial values for the elements

If the element type does not have a default constructor, then we must specify an explicit element initializer along with the size

-  The constructors that take a size are valid *only* for sequential containers; they are not supported for the associative containers

It is worth noting that although we cannot copy or assign objects of built-in array types (§ 3.5.1, p. 114), there is no such restriction on array

```c++
int digs[10] = {0,1,2,3,4,5,6,7,8,9};
int cpy[10] = digs; // error: no copy or assignment for built-in arrays
array<int, 10> digits = {0,1,2,3,4,5,6,7,8,9};
array<int, 10> copy = digits; // ok: so long as array types match
```

###### 9.2.5 Assignment and `swap`

See page. 338 Table 9.4

The sequential containers (except array) also define a member named assign that lets us assign from a different but compatible type, or assign from a subsequence of a container

```c++
list<string> names;
vector<const char*> oldstyle;
names = oldstyle; // error: container types don’t match
// ok: can convert from const char* to string 
names.assign(oldstyle.cbegin(), oldstyle.cend());
```

A second version of assign takes an integral value and an element value

```c++
// equivalent to slist1.clear();
// followed by slist1.insert(slist1.begin(),10,"Hiya!"); 
list<string> slist1(1); // one element, which is the empty 
string slist1.assign(10, "Hiya!"); // ten elements; each one is Hiya!
```

Excepting array, swap does not copy, delete, or insert any elements and is guaranteed to run in constant time

- Unlike how swap behaves for the other containers, swapping two arrays does exchange the elements
- After the swap, pointers, references, and iterators remain bound to the same element they denoted before the swap
- it is best to use the nonmember version of swap (vital for generic programs)

###### 9.2.6 Container Size Operations

 We can use a relational operator to compare two containers only if the appropriate comparison operator is defined for the element type



#### 9.3 Sequential Container Operations

###### 9.3.1 Adding Elements to a Sequential Container

See => page. 343 Table 9.5 for operations

When we use an object to initialize a container, or insert an object into a container, a copy of that object’s value is placed in the container, not the object itself

Element(s) are inserted before the position denoted by the iterator. e.g

```c++
slist.insert(iter,"Hello!");// insert "Hello!" just before iter
// we can take advantage of the return of insert
```

The new standard introduced three new members—`emplace_front`, `emplace`, and `emplace_back`—that construct rather than copy elements

###### 9.3.2 Accessing Elements

See => page. 347 Table 9.6 for operations

The members that access elements in a container (i.e., `front`, `back`, `subscript`, and `at`) return references. If the container is a const object, the return is a reference to const

###### 9.3.3 Erasing Elements

See=> page 349 Table 9.7 for operations

The members that remove elements do not check their argument(s)

`erase` return an iterator referring to the location after the (last) element that was removed

###### 9.3.4 Specialized `forward_list` Operations

See=> page 351 Table 9.8 for operations

`before_begin` returns an off-the-beginning iterator

###### 9.3.5 Resizing a Container

How `resize` work? => see page 352 Table 352 for operations

```c++
list<int> ilist(10, 42);// ten ints: each has value 42 
ilist.resize(15); // adds five elements of value 0 to the back of ilist 
ilist.resize(25, -1);// adds ten elements of value -1 to the back of ilist 
ilist.resize(5); // erases 20 elements from the back of ilist
```

###### 9.3.6 Container Operations May Invalidate Iterators



#### 9.4 How a `vector` Grows

The vector and string types provide members, described in Table 9.10, that let us interact with the memory-allocation part of the implementation

There is no guarantee that a call to `shrink_to_fit` will return memory



#### 9.5 Additional `string` Operations

###### 9.5.1 Other Ways to Construct `string`

=> See Table 9.11

The `substr` function throws an `out_of_range` exception (§ 5.6, p. 193) if the position exceeds the size of the string

- `s.substr(pos, n)`

###### 9.5.2 Other Ways to Change a `string`

=> Table 9.13

###### 9.5.3 `string` Search Operations

6 x 4 search functions => Table 9.14

- Each returns a `string::size_type` value that is the index of where the match occurred
- If there is no match, the function returns a static member named `string::npos`,  unsigned initialized with value -1
- We can pass an optional starting position to the find operations
- `rfind` searches for the last (right-most) occurrence of the indicated substring

###### 9.5.4 The `compare` Functions

=> see Table 9.15

###### 9.5.5 Numeric Conversions

=> see Table 9.16

If the string can’t be converted to a number, These functions throw an `invalid_argument` exception (§ 5.6, p. 193). If the conversion generates a value that can’t be represented, they throw `out_of_range`



#### 9.6 Container Adaptors

an adaptor is a mechanism for making one thing act like another

Table 9.17 lists the operations and types that are common to ALL the container adaptors

By default both `stack` and `queue` are implemented in terms of `deque`, and a `priority_queue` is implemented on a `vector`

- How to override?

  ```c++
  stack<string, vector<string>> str_stk;
  ```

`stack` => see Table 9.18

`queue` & `priority_queue` => see Table 9.19

