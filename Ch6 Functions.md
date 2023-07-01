### Ch 6. Functions

We’ll cover both how to overload functions and how the compiler selects the matching version for a particular call from several overloaded functions

#### 6.1 Function Basics

Although we know which argument initializes which parameter, we have NO guarantees about the order in which arguments are evaluated (§ 4.1.3, p. 137). The compiler is free to evaluate the arguments in whatever order it prefers.

Funtion return type

- may not be an array type (§ 3.5, p. 113) or a function type
- may return a pointer to an array or a function
- We’ll see how to define functions that return pointers (or references) to arrays in § 6.3.3 (p. 228) 
- and how to return pointers to functions in § 6.7 (p. 247).

###### 6.1.1 Local Objects

Objects that exist only while a block is executing are known as **automatic objects**. After execution exits a block, the values of the automatic objects created in that block are undefined.

Local `statics` are not destroyed when a function ends; they are destroyed when the program terminates

```c++
size_t count_calls(){
		static size_t ctr = 0; // value will persist across calls
    return ++ctr;
}
int main() {
		for (size_t i = 0; i != 10; ++i)
  			cout << count_calls() << endl;
		return 0; 
}
// This program will print the numbers from 1 through 10 inclusive
```

###### 6.1.2 Function Declarations

As with variables, a function may be defined only once but may be declared multiple times

- With 1 exception that we’ll cover in § 15.3 (p. 603), we can declare a function that is not defined so long as we never use that function.

Recall that variables are declared in header files and defined in source files. For the same reasons, functions should be declared in header files and defined in source files

###### 6.1.3 Separate Compilation

`Separate compilation` lets us split our programs into several files, each of which can be compiled independently



#### 6.2 Argument Passing

Parameter initialization in function works the same way as variable initialization

###### 6.2.1 Passing Arguments by Value

Programmers accustomed to programming in C often use pointer parameters to access objects outside a function. In C++, programmers generally use reference parameters instead.

###### 6.2.2 Passing Arguments by Reference

As we’ll see in § 6.2.3 (p. 213), functions should use references to `const` for reference parameters they do not need to change

###### 6.2.3 `const` Parameters and Arguments

Just as in any other initialization, when we copy an argument to initialize a parameter, top-level `const` are ignored. As a result, top-level `const` on parameters are ignored. 

- In C++, we can define several different functions that have the same name. How- ever, we can do so only if their parameter lists are sufficiently different. Because top-level consts are ignored, we can pass exactly the same types to either version of fcn. The second version of fcn is an error. Despite appearances, its parameter list doesn’t differ from the list in the first version of fcn.

```c++
void fcn(const int i) { /* fcn can read but not write to i */ } 
void fcn(int i) { /* . . . */ } // error: redefines fcn(int)
```

Because parameters are initialized in the same way that variables are initialized, it can be helpful to remember the general initialization rules. We can initialize an object with a low-level const from a nonconst object but not vice versa, and a plain reference must be initialized from an object of the same type. 

- Exactly the same initialization rules apply to parameter passing

###### 6.2.4 Array Parameters

We cannot copy an array (§ 3.5.1, p. 114), and when we use an array it is (usually) converted to a pointer (§ 3.5.3, p. 117). Because we cannot copy an array, we cannot pass an array by value. Because arrays are converted to pointers, when we pass an array to a function, we are actually passing a pointer to the array’s first element.

We’ll see in § 16.1.1 (p. 654) how we might write this function in a way that would allow us to pass a reference parameter to an array of any size

How to pass a 2d array?

```c++
void print(int (*matrix)[10], int rowSize) { /* . . . */ }
```

###### 6.2.5 `main:` Handling Command-Line Options

`argv`, is an array of pointers to C-style character strings

```c++
int main(int argc, char *argv[]) { ... }
prog -d -o ofile data0 // will get
argv[0] = "prog"; // or argv[0] might point to an empty string 
argv[1] = "-d";
argv[2] = "-o";
argv[3] = "ofile";
argv[4] = "data0";
argv[5] = 0; // The element just past the last pointer is guaranteed to be 0
```

###### 6.2.6 Functions with Varying Parameters

C++11 has 2 ways to handle varying number of arguments

- If all arguments have same type => `initializer_list`

  See page. 220 Table 6.1 for operations

- If not => see variadic template in 16.4 (p. 699)

`ellipsis parameters`: `...` 

- NO type cheking
- `...` must appear last
- e.g. `void foo(parm_list, ...);`



#### 6.3 Return Types and the `return` Statement

###### 6.3.2 Functions That Return a Value

Values are returned in exactly the SAME way as variables and parameters are initialized

- The return value is used to initialize a temporary at the call site, and that temporary is the result of the function call
- C++11 can use `{}` to list initialize the temporary
- NEVER return a reference/pointer to a local object

1 exception: `main` don't need a `return` (compiler will add one for us)

- defined in `cstdlib` we have `return EXIT_FAILURE` and `return EXIT_SUCCESS`

###### 6.3.3 Returning a Pointer to an Array

Return type complicated => Make life easier => using `trailing return type` OR `decltype`

```c++
auto func(int i) -> int(*)[10];
decltype(xxx) *fun();
// instead of 
int (*func(int))[10];
```

- The only tricky part is that we must remem- ber that decltype does not automatically convert an array to its corresponding pointer type



#### 6.4 Overloaded Functions

Overloaded functions must differ in the number or the type(s) of their parameters

- two functions ONLY return types differ => ERROR

  ```c++
  Record lookup(const Account&);
  bool lookup(const Account&);
  ```

- parameter names are ignored

  ```c++
  Record lookup(const Account &acct);
  Record lookup(const Account&);
  ```

- `Telno` and `Phone` are the same type

  ```c++
  typedef Phone Telno;
  Record lookup(const Phone&);
  Record lookup(const Telno&);
  ```

- As we saw in § 6.2.3 (p. 212), top-level `const` has no effect on the objects that can be passed to the function

  ```c++
  Record lookup(Phone);
  Record lookup(const Phone);
  Record lookup(Phone*);
  Record lookup(Phone* const);
  ```

- Low-level `const` is OK

  ```c++
  Record lookup(Account&); 
  Record lookup(constAccount&); // override works
  Record lookup(Account*);
  Record lookup(const Account*); // override works
  ```

Compiler can use low-level `const` to distinguish which function to call

- NO conversion from `const` (§ 4.11.2, p. 162)
- Conversion to const OK
- BUT, as we’ll see in § 6.6.1 (p. 246), the compiler will prefer the `nonconst` versions when we pass a nonconst object or pointer to `nonconst`

Compiler determines which function to call by comparing the args in the call with the parameters offered by each function in the overload set 

- Tricky when same number of parameters and conversions happen
- 3 possibilities:
- Case 1: Exactly 1 function that is best match => call that
- Case 2: No match => error
- Case 3: More than 1 function that matches and none of them is clearly best => Error, Ambiguous call

###### 6.4.1 Overloading and Scope

Ordinarily, it is a bad idea to declare a function locally. However, to explain how scope interacts with overloading, we will violate this practice and use local function declarations

If we declare a name in an inner scope, that name HIDES uses of that name declared in an outer scope. Names do not overload across scopes

- In C++, name lookup happens before type checking

```c++
string read();
void print(const string &);
void print(double);
void fooBar(int ival){
  bool read = false; // new scope: hides the outer declaration of read 
  string s = read(); // error: read is a bool variable, not a function
  void print(int); // new scope: hides both previous instances of print
  print("Value:"); //  error: print(const string &) is hidden
  print(ival); // ok: print(int) is visible
  print(3.14); // ok: calls print(int); print(double) is hidden
}
```



#### 6.5 Features for Specialized Uses

###### 6.5.1 Default Arguments

If a parameter has a default argument, all the parameters that follow it MUST also have default arguments

```c++
typedef string::size_type sz;  
string screen(sz ht = 24, sz wid = 80, char backgrnd = ’ ’);
```

Can omit only trailing (right-most) arguments

```c++
window = screen(, , ’?’); // error 
```

Part of the work of designing a function with default arguments is ordering the parameters so that those least likely to use a default value appear first and those most likely to use a default appear last

It is OK to add more default args

```c++
string screen(sz, sz, char = ’ ’);
string screen(sz, sz, char = ’*’); // error: we cannot change an already declared default value
string screen(sz = 24, sz = 80, char); // ok: adds default arguments
```

Names used as default arguments are resolved in the scope of the function declaration

- THUS Local variables may NOT be used as a default argument

  ```c++
  sz wd = 80;
  char def = ’ ’;
  sz ht();
  string screen(sz = ht(), sz = wd, char = def); 
  string window = screen(); // calls screen(ht(),80,’’)
  void f2() {
    def = ’*’; // changes the value of a default argument
    sz wd = 100; // hides the outer definition of wd but does not change the default
    window = screen(); // calls screen(ht(),80,’*’) 
  }
  ```

###### 6.5.2 Inline and `constexpr` Funtions

`inline` function can avoid function call overhead

- Via expand during compilation
-  The inline specification is ONLY a request to the compiler. The compiler may choose to ignore this request
- Many compilers will not inline a recursive function
- A 75-line function will almost surely not be expanded inline

`constexpr` function

- The return type and the type of each parameter MUST be a literal type
- Function body MUST contain exactly one return statement

In order to be able to expand the function immediately, `constexpr` functions are implicitly `inline`

A `constexpr` function is permitted to return a value that is not a constant

```c++
// scale(arg) is a constant expression if arg is a constant expression
constexpr size_t scale(size_t cnt) { return new_sz() * cnt; }
```

Unlike other functions, `inline` and `constexpr` functions may be defined multiple times in the program. However, all of the definitions of a given `inline` or `constexpr` must match exactly. As a result, `inline` and `constexpr` functions normally are defined in headers.

###### 6.5.3 Aids for Debugging

`assert` is a preprocessor macro, do runtime check

We can “turn off” debugging by 

- providing a `#define` to define `NDEBUG`
- OR `g++ -D NDEBUG main.cpp`

The compiler defines `__func__` in every function. It is a local static array of const char that holds the name of the function

- `__FILE__` string literal containing the name of the file 
- `__LINE__` integer literal containing the current line number
- `__TIME__` string literal containing the time the file was compiled
- `__DATE__` string literal containing the date the file was compiled



#### 6.6 Function Matching

Step 1: identify candidate functions (same name and declaration is visible)

Step 2: identify viable function (same number of parameters, type of each must match or be convertible)

Step 3: the closer the types of the argument and parameter are to each other, the better the match 

```c++
void f();
void f(int);
void f(int, int);
void f(double, double = 3.14); 
f(5.6); // calls void f(double,double)
// step 1, all remains
// step 2, only f(int) and f(double, double = 3.14) remains
// step 3, f(double, double = 3.14) since no conversion need
f(42, 2.56) // rejected by compiler because ambiguous
```

###### 6.6.1 Arg Type Conversions

Conversion rank => 

1. Exact match

   - Identical type
   - Arg converted from array/function to pointer => see § 6.7 (p. 247) function pointers
   - Top-level `const` difference does not matter

2. `const` conversion => §4.11.2,p.162

3. Promotion => § 4.11.1, p. 160

   ```c++
   void ff(int);
   void ff(short);
   ff(’a’); // char promotes to int; calls f(int)
   
   void manip(long);
   void manip(float); 
   manip(3.14); // error: ambiguous call
   ```

4. Arithmetic/pointer conversion => § 4.11.1, p. 159, § 4.11.2, p. 161

5. Class-type conversion => see § 14.9 p. 579



#### 6.7 Pointers to Functions

```c++ 
bool lengthCompare(const string &, const string &);
bool (*pf)(const string &, const string &); // uninitialized
// () is MUST, otherwise
// declares a function named pf that returns a bool* 
bool *pf(const string &, const string &);
```

When we use the name of a function as a value, the function is automatically converted to a pointer

```c++
pf = lengthCompare;
pf = &lengthCompare; // same
```

Moreover, we can use a pointer to a function to call the function to which the pointer points, NO need to dereference

```c++
bool b1 = pf("hello", "goodbye"); // calls lengthCompare 
bool b2 = (*pf)("hello", "goodbye"); // equivalent call
bool b3 = lengthCompare("hello", "goodbye"); // equivalent call
```

When we pass a function as an argument, we can do so directly. It will be automatically converted to a pointer, BUT the return type is not automatically converted to a pointer type

```c++
void useBigger(const string &s1, const string &s2, bool pf(const string &, const string &));
// same
void useBigger(const string &s1, const string &s2, bool (*pf)(const string &, const string &));
```

Use `auto` and `decltype` to make life easier

- The only tricky part is when we apply decltype to a function, it returns a function type, not a pointer to function type. We must add a `*` to indicate that we are returning a pointer, not a function

