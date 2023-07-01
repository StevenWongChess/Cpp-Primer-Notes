### Ch 2. Variables and Basic Types

#### 2.1 Primitive Built-in Types

###### 2.1.1 Arithmetic Types

C++ is a statically typed language, type checking is done at compile time

Most compilers provide more precision than the specified minimum

4 rules to choose type:

- use more `unsigned`
- DO NOT overuse `char` (specify `unsigned` or `signed`)
- use more `int`, `double`
- DO NOT mix `unsigned` with `signed`

###### 2.1.2 Type Conversions

Compiler automatically does `type conversion` when another type is expected (more to say in § 4.11 page. 159)



```c++
bool b = 42; // b is true, 0 => false, other => true
int i = b; // i is 1, true => 1, false => 0
i = 3.14; // i is 3
double pi = i; // pi is 3.0
unsigned char c = -1; // assuming 8-bit chars, c is 255
signed char c2 = 256; // assuming 8-bit chars, c2 undefined
```

###### 2.1.3 Literals

The compiler appends a null character `\0` to every string literal. Thus, the actual size of a string literal is 1 more

Two string literals that appear adjacent to one another and that are separated only by spaces, tabs, or newlines are concatenated into a single literal

- i.e. `"A" "B" is just "AB"`

Escape sequence: if a `\` is followed by more than 3 octal digits, ONLY the first 3 are associated with the `\`

How to specify the Type of a Literal => page.40



#### 2.2 Variable

###### 2.2.1 Variable Definition

`initialization != assignment`

`list initialization`: `int x{0};`

- Loss of information is not allowed by compiler for {} built-in type

  ```c++
  int x{3.14}; // error
  ```

When we define a variable without an initializer, the variable is `default initialized`

Variables of built-in type defined outside any function body are initialized to zero

- 1 Exception in § 6.1.1 (page. 205)!!! ... defined inside a function are `uninitialized`

###### 2.2.2 Variable Declaration and Definition

To support `separate compilation`, C++ distinguishes between

- `declarations` = makes a name known to the program (A file that wants to use a name defined elsewhere includes a declaration for that name)
- AND  `definitions` = creates the associated entity 

It is an error to provide an initializer on an extern inside a function

Variables must be defined exactly once but can be declared many times

```c++
extern int i; // declares but does not define i 
int j; // declares and defines j
extern double pi = 3.1416; // definition
```

###### 2.2.3 Identifiers

`Identifiers` naming rules & C++ keywords=> page. 46-47

###### 2.2.4 Scope of a Name

Scope (The global scope has no name `::`)

```c++
int reused = 42; // reused has global scope
int main() {
	int unique = 0; // unique has block scope 
	std::cout << reused << " " << unique << std::endl; 
  // output#1: uses global reused; prints 42 0
  int reused = 0;
  // new,local object named reused hides global reused
	std::cout << reused << " " << unique << std::endl;
  // output#2: uses local reused; prints 0 0
  std::cout << ::reused << " " << unique << std::endl;
  // output#3: explicitly requests the global reused; prints 42 0 
	return 0; 
}
```



#### 2.3 Compound Types

###### 2.3.1 References

`references` MUST be initialized

- Because no way to rebind a reference
- but pointer do NOT need 

```c++
int &refVal2; // error: a reference must be initialized
```

`Reference` is NOT an object, it is just another name for an already existing object, BUT pointer is an object

With two exceptions that we’ll cover in 

- § 2.4.1-2 (p. 61-62) 
- and § 15.2.3 (p. 601), 
- the type of a reference and the object to which the reference refers MUST match exactly (same for pointer)

```c++
double dval = 3.14;
int &refVal5 = dval;// error: initializer must be an int object
```

Reference may be bound only to 

- an object
- not to a literal or to the result of a more general expression

```c++
int &refVal4 = 10; // error: initializer must be an object
```

###### 2.3.2 Pointers

Preprocessor variables are managed by the preprocessor, and are not part of the std namespace

- As a result, we refer to them directly without the std:: prefix.

Generally, we use a `void*` pointer to deal with memory as memory, rather than using the pointer to access the object stored in that memory. We’ll cover using `void*` pointers in this way in 

- § 19.1.1 (p. 821). 
- § 4.11.3 (p. 163) will show how we can retrieve the address stored in a `void*` pointer

###### 2.3.3 Understanding Compound Type Declarations

The easiest way to understand the type of `r` is to read the definition right to left `e.g. int *&r` is reference to pointer



#### 2.4 `const` Qualifier

Because we can’t change the value of a `const` object after we create it, it must be initialized (think about reference)

By Default, const Objects Are Local to a File 

- (thanks to `inline` in C++17 no longer a problem any more) 
- (detail in page. 60, HARD!!! https://www.reddit.com/r/cpp_questions/comments/c96pjq/understanding_const_variables_being_local_to_a/)

###### 2.4.1 References to `const`

A `reference to const` cannot be used to change the object to which the reference is bound

```c++
const int ci = 1024;
const int &r1 =  ci; // ok: both reference and underlying object are const
r1 = 42; // error: r1 is a reference to const
int &r2 = ci; // error: nonconst reference to a const object, 
// If this initialization were legal, we could use r2 to change the value of its underlying object.
```

In § 2.3.1 (p. 51) we noted that there are two exceptions to the rule that the type of a reference must match the type of the object to which it refers. 

- FIRST EXCEPTION:  we can initialize a reference to const from any expression that can be converted to the type of the reference. 
- In particular, we can bind a reference to const to a nonconst object, a literal, or a more general expression.

```c++
int i = 42;
const int &r1 = i; // we can bind a const int& to a plain int object 
const int &r2 = 42; // ok: r1 is a reference to const 
const int &r3 = r1 * 2;// ok: r3 is a reference to const 
int &r4 = r * 2; // error: r4 is a plain, nonconst reference
```

###### 2.4.2 Pointers and `const`

In § 2.3.2 (p. 52) we noted that there are two exceptions to the rule that the types of a pointer and the object to which it points must match

- FIRST EXCEPTION: we can use a pointer to const to point to a nonconst object

###### 2.4.3 Top-Level `const`

`Top-level const` vs `low-level const`

We can convert a nonconst to const but not the other way round

```c++
const int *const p = xxx;
int *newp = p; // Error!
```

###### 2.4.4 `constexpr` and Constant Expressions

`constexpr` = const + can be evaluated at compile time; 

- we’ll see in § 6.5.2 (p. 239) that the new standard lets us define certain functions as `constexpr`
- `constexpr` specifier applies to the pointer, not the type to which the pointer points

`Literal types` = arithmetic, reference, pointer, § 7.5.6 (p. 299) and § 19.3 (p. 832)



#### 2.5 Dealing with Types

###### 2.5.1 Type Aliases

`typedef` => `using` => `auto`

```c++
typedef int alis;
using alis = int;
```

###### 2.5.2 `auto` Type Specifier

By implication, a variable that uses auto as its type specifier must have an initializer

Because a declaration can involve only a single base type, the initializers for all the variables in the declaration must have types that are consistent with each other

```c++
auto i = 0, *p = &i; // ok: i is int and p is a pointer to int 
auto sz = 0, pi = 3.14; // error: inconsistent types for sz and pi
```

The type that the compiler infers for auto is not always exactly the same as the initializer’s type. Instead, the compiler adjusts the type to conform to normal ini- tialization rules.

- FIRST, when we use a reference as an initializer, the initializer is the corresponding object

  ```c++
  int i = 0, &r = i;
  auto a = r; // a is an int(r is an alias for i, which has type int)
  ```

- SECOND, `auto` ignore top-level `const`, low-level `const` is kept

  ```c++
  const int ci = i, &cr = ci;
  auto b = ci; // b is an int (top-level const in ci is dropped)
  auto c = cr; // c is an int (cr is an alias for ci whose const is top-level) 
  auto d = &i; // d is an int*(& of an int object is int*)
  auto e = &ci; // e is const int* (& of a const object is low-level const)
  ```

- If we want the deduced type to have a top-level const, we must say so explicitly

  ```c++
  const auto f = ci; // deduced type of ci is int; f has type const int
  ```

- We can also specify that we want a reference to the auto-deduced type. Normal initialization rules still apply

  ```c++
  auto &g = ci; // g is a const int& that is bound to ci 
  auto &h = 42; // error: we can’t bind a plain reference to a literal const 
  auto &j = 42; // ok: we can bind a const reference to a literal
  ```

- When we ask for a reference to an auto-deduced type, top-level consts in the initializer are not ignored.

  ```c++
  auto k = ci, &l = i; // k is int; l is int&
  auto &m = ci, *p = &ci; // m is a const int&; p is a pointer to const int 
  // error:type deduced from i is int; type deduced from &ci is const int
  auto &n = i, *p2 = &ci;
  ```

###### 2.5.3 `decltype` Type Specifier

`decltype` : The compiler analyzes but does NOT evaluate the expression

The way decltype handles top-level const and references differs subtly from the way auto does

- Case 1: `decltype(variable)` => returns the type of that variable, including top-level const and references

  ```c++
  const int ci = 0, &cj = ci;
  decltype(ci) x = 0; // x has type const int
  decltype(cj) y = x; // y has type const int& and is bound to x
  decltype(cj) z; // error: z is a reference and must be initialized
  ```

  It is worth noting that `decltype` is the **ONLY** context in which a variable defined as a reference is not treated as a synonym for the object to which it refers

- Case 2: `decltype(expression that is not variable)` => returns the type that the expression yields

  ```c++
  // decltype of an expression can be a reference type
  int i = 42, *p = &i, &r = i;
  decltype(r + 0) b; // ok: addition yields an int; b is an (uninitialized) int 
  decltype(*p) c; // error:c is int& and must be initialized
  ```

  Generally speaking, `decltype` returns a reference type for expressions that yield objects that can stand on the left-hand side of the assignment, see more in § 4.1.1 (p. 135)

  `decltype(r)` return reference type, `decltype(*p)` return reference type

- Case 3: `()` makes variable an expression, `decltype((variable))` return reference type

  ```c++
  decltype ((i)) d; // error: d is int& and must be initialized 
  decltype (i) e; // ok:e is an (uninitialized) int
  ```



#### 2.6 Defining Our Own Data Structures

###### 2.6.3 Writing Our Own Header Files

Classes ordinarily are not defined inside functions

- Exception: We’ll see in § 19.7 (p. 852), we can define a class inside a function, such classes have limited functionality

Whenever a header is updated, the source files that use that header must be recompiled to get the new or changed declarations

Preprocessor & header guard

```c++
#ifndef SALES_DATA_H
#define SALES_DATA_H
Stuff here
#endif
```

- Preprocessor variable names do not respect C++ scoping rules
- To avoid name clashes with other entities in our programs, preprocessor variables usually are written in all uppercase









