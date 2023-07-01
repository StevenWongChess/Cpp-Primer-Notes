### Ch 3. Strings, Vector, and Arrays

#### 3.1 Namespace using Declarations

There are easier ways to use namespace members. The safest way is a **using declaration**. § 18.2.2 (p. 793) covers another way to use names from a namespace

- e.g. `using std::cin;`
- Code inside headers ordinarily should not use using declarations



#### 3.2 Library `string` Type

###### 3.2.2 Operations on `string`

`string` operations => page.86 Table 3.2

The string input operator `>>` 

- Reads and discards any leading whitespace (e.g., spaces, newlines, tabs). 
- It then reads characters until the next whitespace character is encountered

`getline` 

- reads the given stream up to and including the first newline 
- and stores what it read except the newline `\n`

The string class—and most other library types—defines several companion types

- e.g. `string::size_type`
- `size` returns an unsigned, type DO NOT mix with signed stuff

`string` comparison

- first length
- then char by char

For historical reasons, and for compatibility with C, string literals are *not* standard library strings

- Can’t add string literals 
- e.g. `string s5 = "hello" + ","; // error:no string operand`

###### 3.2.3 Dealing with the Chars in `string`

Use the C++ versions of C library headers

- C Library `name.h` <=> C++ Library `cname`

`cctype` Functions => page.92 Table 3.3



#### 3.3 Library `vector` Type

The process that the compiler uses to create classes or functions from templates is called `instantiation`

Old style ` vector<vector<int> >`, after C++11 => ` vector<vector<int>>`

###### 3.3.1 Defining and Initializing `vector`

If vector hold object of type that has default initializer, we can supply size only. Otherwise explicit initializer is needed

Ways to initialize a `vector` => page.99 Table 3.4

- `{}`: Only if it is not possible to list initialize the object will the other ways to initialize the object be considered

- `()`: On the other hand, if we use braces and there is no way to use the initializers to list initialize the object, then those values will be used to construct the object

###### 3.3.2 Adding Elements to a `vector`

We cannot use a `range for` if the body of the loop adds elements to the vector

- Reasons in § 5.4.3 (page. 188)



#### 3.4 Introducing Iterators

Why iterators?

- All of the library containers have iterators, but only a few of them support the subscript operator

###### 3.4.1 Using Iterators

If the container is empty => `begin` = `end` = both off-the-end iterators

A `const_iterator` behaves like a const pointer (§ 2.4.2, p. 62); If a vector or string is const, we may use only its `const_iterator` type

- C++11 => `cbegin` and `cend`

Arrow operator = dereference + member

- i.e. `it->mem` is a synonym for `(*it).mem`

###### 3.4.2 Iterator Arithmetic

`it1 - it2` returns a `difference_type`



#### 3.5 Arrays

###### 3.5.1 Defining and Initializing Built-in Arrays

The dimension must be known at compile time (be constant expression)

```c++
unsigned cnt = 42; // not a constant expression 
constexpr unsigned sz = 42; // constant expression
intarr[10]; // array of ten ints
int *parr[sz]; // array of 42 pointers to int
stringbad[cnt]; // error:cnt is not a constant expression
stringstrs[get_size()];// ok if get_size is constexpr,error otherwise
```

If we omit the dimension, the compiler infers it from the number of initializers

If the dimension is greater than the number of initializers, the initializers are used for the first elements and any remaining elements are value initialized

```c++
int a2[] = {0, 1, 2}; // an array of dimension 3
string a4[3]={"hi","bye"}; // sameasa4[]={"hi","bye",""}
int a5[2] = {0,1,2}; // error: too many initializers
```

`char array` are special

```c++
char a1[] = {’C’, ’+’, ’+’}; // list initialization, no null
char a3[] = "C++"; // null terminator added automatically
const char a4[6] = "Daniel"; // error: no space for the null!
```

NO `copy/assignment`! Some compilers allow array assignment as a `compiler extension`

It can be easier to read array declarations from the inside out rather than from right to left

```c++
int *ptrs[10]; // ptrs is an array of ten pointers to int 
int &refs[10] = /* ? */; // error: no arrays of references
int (*Parray)[10] = &arr; // Parray points to an array of ten ints 
int (&arrRef)[10] = arr; // arrRef refers to an array of ten ints
int *(&arry)[10] = ptrs; // arry is a reference to an array of ten pointers
```

###### 3.5.3 Pointers and Arrays

In C++ pointers and arrays are closely intertwined

- In most expressions, when we use an object of array type, we are really using a pointer to the first element in that array

  ```c++
  string *p2 = nums; // equivalent to p2 = &nums[0]
  ```

- There are various implications of the fact that operations on arrays are often really operations on pointers. One such implication is that when we use an array as an initializer for a variable defined using auto (§ 2.5.2, p. 68), the deduced type is a pointer, not an array

  ```c++
  int ia[] = {0,1,2,3,4,5,6,7,8,9}; // ia is an array of ten ints
  auto ia2(ia); // ia2 is an int* that points to the first element in ia 
  ia2 = 42; // error: ia2 is a pointer, and we can’t assign an int to a pointer
  
  auto ia2(&ia[0]); // now it’s clear that ia2 has type int*
  ```

- It is worth noting that this conversion does not happen when we use decltype. The type returned by `decltype(ia)` is array of ten ints:

  ```c++
  // ia3 is an array of ten ints
  decltype(ia) ia3 = {0,1,2,3,4,5,6,7,8,9};
  ia3 = p; // error: can’t assign an int* to an array 
  ia3[4] = i; // ok: assigns the value of i to an element in ia3
  ```

Pointers that address elements in an array have additional operations beyond those we described in § 2.3.2 (p. 52)

- `begin(arr)`, `end(arr)`
- Pointers that address array elements can use all the iterator operations listed in Table3.6(p.107)andTable3.7(p.111)
- The result of subtracting two pointers is a library type named `ptrdiff_t`

Unlike subscripts for vector and string, the index of the built-in subscript operator is not an unsigned type

```c++
int *p = &ia[2]; // p points to the element indexed by 2 
int j = p[1]; // p[1] is equivalent to *(p + 1),
							// p[1] is the same element as ia[3] 
int k = p[-2]; // p[-2] is the same element as ia[0]
```

###### 3.5.4 C-Style Char Strings

C strings are SHIT! => see `3.5.4`

###### 3.5.5 Mix Older Code

How to mix them?

- We can use a null-terminated character array anywhere that we can use a string literal
- The reverse functionality is not provided: There is no direct way to use a library string when a C-style string is required

Cannot initialize array with array OR vector, BUT can initialize vector with array 

```c++
int int_arr[] = {0, 1, 2, 3, 4, 5};
// ivec has six elements; each is a copy of the corresponding element in int_arr 
vector<int> ivec(begin(int_arr), end(int_arr));
// copies three elements: int_arr[1],int_arr[2],int_arr[3]
vector<int> subVec(int_arr + 1, int_arr + 4);
```

Modern C++ programs should use `vectors` and `iterators` instead of built-in `arrays` and `pointers`, and use `strings` rather than C-style array-based `character strings`



#### 3.6 n-d Arrays

Strictly speaking, there are no multidimensional arrays in C++. What are commonly referred to as multidimensional arrays are actually arrays of arrays

Treat this section as dictionary to look up when needed!

