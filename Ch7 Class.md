### Ch 7. Class

In Chapter 13 we’ll learn how to control what happens when objects are copied, moved, assigned, or destroyed. In Chapter 14 we’ll learn how to define our own operators

#### 7.1 Defining Abstract Data Types

###### 7.1.2 Defining the Revised `Sales_data` Class

Functions defined in the class are implicitly inline (§ 6.5.2, p. 238)

With one exception that we’ll cover in § 7.6 (p. 300), when we call a member function we do so on behalf of an object

The purpose of `const` member function is to modify the type of the implicit `this` pointer

- Objects that are const, and references or pointers to const objects, may call only const member functions
- our function would be more flexible if this were a pointer to const
- However, this is implicit and does not appear in the parameter list. There is no place to indicate that this should be a pointer to const. The language resolves this problem by letting us put const after the parameter list of a member function

```c++
std::string isbn() const { return this->bookNo; }
// imagine const work as below (not legal)
std::string Sales_data::isbn(const Sales_data *const this)
```

###### 7.1.3 Defining Nonmember Class-Related Functions

Ordinarily, nonmember functions that are part of the interface of a class should be declared in the same header as the class itself

###### 7.1.4 Constructors

In this section, we’ll introduce the basics of how to define a constructor. Constructors are a surprisingly complex topic. Indeed, we’ll have more to say about constructors in 

- § 7.5 (p. 288), 
- § 15.7 (p. 622)
- § 18.1.3 (p. 777)
- in Chapter 13

The compiler generates a default constructor (known as `synthesized default constructor`) automatically ONLY if a class declares NO constructors

Some Class CANNOT rely on synthesized default constructor

- If we define any other constructors, compiler won't generate 

  Solution: use `class_name() = default;`

- Generate wrong thing

- Compiler unable to synthesize. e.g. If a class has a member that has a class type and that class does not have a default constructor

  We’ll see in § 13.1.6 (p. 508) additional circumstances that prevent the compiler from generating an appropriate default constructor

When a member is omitted from the constructor initializer list, it is im- plicitly initialized using the same process as is used by the synthesized default constructor.

###### 7.1.5 Copy, Assignment, and Destruction

We’ll show how we can define our own versions of these operations in Chapter 13 

Again compiler synthesized, again some class CANNOT rely on synthesized versions

- classes that allocate resources that reside outside the class objects themselves

  Solution: use `stl`



#### 7.2 Access Control and Encapsulation

###### 7.2.1 Friends

A class makes a function its friend by including a declaration for that function preceded by the keyword friend

- `friend` declaration only specifies access
- MUST also declare the function separately from the `friend` declaration
- To make a friend visible to users of the class, we usually declare each friend (outside the class) in the same header as the class itself

We’ll have more to say about friendship in § 7.3.4 (p. 279)



#### 7.3 Additional Class Features

 These features include `type members`, `in-class initializers for members of class type`, `mutable data members`, `inline member functions`, `returning *this from a member function`, `more about how we define and use class types`, and `class friendship`

###### 7.3.1 Class Members Revisited

`Type Member` (often appear at top)

- Use `typedef` OR `using`
- For reasons we’ll explain in § 7.4.1 (p. 284), unlike ordinary members, members that define types must appear before they are used. As a result, type members usually appear at the beginning of the class

`inline`

- As we’ve seen, member functions defined inside the class are automatically `inline` (§ 6.5.2, p. 238)
- We can explicitly declare a member function as `inline`
  - Either declaration inside class body
  - OR function definition outside the class body
  - Legal to do both, BUT do latter only is more easier to read

A `mutable` data member is never const, even when it is a member of a const object

###### 7.2.3 Functions that Return `*this`

 A const member function that returns `*this` as a reference should have a return type that is a reference to const

###### 7.3.3 Class Types

Two different classes define two different types even if they define the same members

Can declare a class without defining it (forward definition) => page. 279

- That class is `incomplete type`
- ONLY limited ways to use

###### 7.3.4 Friendship Revisited

`friend` to whole class

```c++
class Screen {
		// Window_mgr members can access the private parts of class Screen 
	  friend class Window_mgr;
		// ...rest of the Screen class
};
```

- NOTE: friendship is not transitive

`friend` to only one member function

```c++
class Screen {
		// Window_mgr::clear must have been declared before class Screen 
  	friend void Window_mgr::clear(ScreenIndex);
		// ...rest of the Screen class
};
```

Although overloaded functions share a common name, they are still different functions

- a class must declare as a friend each function in a set of overloaded functions that it wishes to make a friend

Even if we define the function inside the class, we must still provide a declaration outside of the class itself to make that function visible. A declaration must exist even if we only call the friend from members of the friendship granting class

```c++
struct X {
		friend void f() { /* friend function can be defined in the class body */ } 
		X() { f(); } // error: no declaration for f
		void g();
		void h();
};
void X::g(){return f();}// error: f hasn’t been declared
void f(); // declares the function defined inside X 
void X::h() { return f(); } // ok: declaration for f is now in scope
```



#### 7.4 Class Scope

Function name and return type MUST include `class_name::`

###### 7.4.1 Name Lookup and Class Scope

Member function definitions are processed after the compiler processes all of the declarations in the class.

- applies only to names used in the body of a member function

If a member declaration uses a name that has not yet been seen inside the class, the compiler will look for that name in the scope(s) in which the class is defined

```c++
typedef double Money;
string bal;
class Account {
public:
	Money balance() { return bal; } // Money is outside, while bal is inside
private:
	Money bal;
// ... };
```

In a class, if a member uses a name from an outer scope and that name is a type, then the class may not subsequently redefine that name

```c++
typedef double Money;
class Account {
public:
Moneybalance(){returnbal;} // uses Money from the outer scope 
private:
	typedef double Money; // error: cannot redefine Money 
  Money bal;
// ...
};

```

- Although it is an error to redefine a type name, compilers are not required to diagnose this error

Name used in body of member function ,order of scope lookup TRICKY!!!

-  first find in member function
- then in class (to specify `this->var`)
- outside of class `(::var)`

When a member is defined outside its class, the third step of name lookup includes names declared in the scope of the member definition as well as those that appear in the scope of the class definition

```c++
int height; // defines a name subsequently used inside Screen 
class Screen {
public:
  typedef std::string::size_type pos;
  void setHeight(pos);
  pos height = 0; // hides the declaration of height in the outer scope
};
Screen::pos verify(Screen::pos); 
void Screen::setHeight(pos var) {
// var: refers to the parameter
// height: refers to the class member 
// verify: refers to the global function 
  height = verify(var);
}
```



#### 7.5 Constructors Revisited

We covered the basics of construtors in § 7.1.4 (p. 262). In this section we’ll cover some additional capabilities of constructors, and deepen our coverage of the material introduced earlier

###### 7.5.1 Constructor Initializer List

Sometimes constructor initializers are required

- The essence of assignment is default initialize first then assignment, less efficient
- Members that are const or references must be initialized
- Similarly, members that are of a class type that does not define a default constructor also must be initialized

Constructor initializer list specifies only the values used to initialize the members, not the order in which those initializations are performed.

- Members are initialized in the order in which they appear in the class definition

 A constructor that supplies default arguments for all its parameters also defines the default constructor

###### 7.5.2 Delegating Constructors

A delegating constructor uses another constructor from its own class to perform its initialization (Similar to Swift)

- When a constructor delegates to another constructor, the constructor initializer list and function body of the delegated-to constructor are both executed

```c++
class Sales_data {
public:
		// nondelegating constructor initializes members from corresponding arguments 
  	Sales_data(std::string s, unsigned cnt, double price): bookNo(s), 			
  			units_sold(cnt), revenue(cnt*price) { } 
  	// remaining constructors all delegate to another constructor
		Sales_data(): Sales_data("", 0, 0) {} 
  	Sales_data(std::string s): Sales_data(s, 0,0) {} 
  	Sales_data(std::istream &is): Sales_data() { read(is, *this); }
		// other members as before }
;
```

###### 7.5.3 The Role of the Default Constructor

The default constructor is used automatically whenever an object is default or value initialized

- Default initialization happens
  - When we define `nonstatic` variables (§2.2.1,p.43) or arrays (§3.5.1,p.114) at block scope without initializers
  - When a class that itself has members of class type uses the synthesized default constructor (§ 7.1.4, p. 262)
  - When members of class type are not explicitly initialized in a constructor initializer list (§ 7.1.4, p. 265)
- Value initialization happens
  - During array initialization when we provide fewer initializers than the size of the array (§ 3.5.1, p. 114)
  - When we define a local static object without an initializer (§ 6.1.1, p. 205)
  - When we explicitly request value initialization by writing an expressions of the form T() where T is the name of a type (The vector constructor that takes a single argument to specify the vector’s size (§ 3.3.1, p. 98) uses an argument of this kind to value initialize its element initializer.)

###### 7.5.4 Implicit Class-Type Conversions

We’ll see in § 14.9 (p. 579) how to define conversions from a class type to another type.

 A constructor that can be called with a single argument defines an implicit conversion from the constructor’s parameter type to the class type

- The `Sales_data` constructors that take a string and that take an istream both define implicit conversions from those types to `Sales_data`. That is, we can use a string or an istream where an object of type `Sales_data` is expected
- Only 1 class-type conversion is allowed

```c++
// error: requires two user-defined conversions:
// (1) convert "9-999-99999-9" to string
// (2) convert that (temporary) string to Sales_data 
item.combine("9-999-99999-9");
```

We can prevent the use of a constructor in a context that requires an implicit conversion by declaring the constructor as `explicit`

- The `explicit` keyword is meaningful ONLY on constructors that can be called with 1 argument
- `explicit` constructors can be used ONLY for direct initialization

```c++
Sales_data item1(null_book); // ok: direct initialization
// error: cannot use the copy form of initialization with an explicit constructor 
Sales_data item2 = null_book;
```

###### 7.5.5 Aggregate Classes

A class is an aggregate if (BAD, do not use aggregate)

- All of its data members are `public`
- It does not define any constructors
- It has no in-class initializers
- It has no base classes or virtual functions, which are class-related features that we’ll cover in Chapter 15

###### 7.5.6 Literal Classes

Unlike other classes, classes that are literal types may have function members that are `constexpr`

An `aggregate` class whose data members are all of literal type is a literal class

A `nonaggregate` class, that meets the following restrictions, is ALSO a literal class

- The data members all must have literal type
- The class must have at least one `constexpr` constructor
- If a data member has an in-class initializer, the initializer for a member of built-in type must be a constant expression, or if the member has class type, the initializer must use the member’s own constexpr constructor
- The class must use default definition for its destructor, which is the member that destroys objects of the class type

A `constexpr` constructor can be declared as `= default` (or as a deleted function, which we cover in § 13.1.6 (p. 507))

Otherwise a `constexpr` constructor must meet the requirements of a constructor—meaning it can have no return statement—and of a `constexpr` function—meaning the only executable statement it can have is a `return` statement. As a result, the body of a `constexpr` constructor is typically empty. We define a `constexpr` constructor by preceding its declaration with the keyword `constexpr`

A `constexpr` constructor MUST initialize every data member. The initializers must either use a `constexpr` constructor or be a constant expression

A `constexpr` constructor is used to generate objects that are `constexpr` and for parameters or return types in `constexpr` functions



#### 7.6 `static` Class Members

Classes sometimes need members that are associated with the class, rather than with individual objects of the class type

- e.g. a bank account class might need a data member to represent the current prime interest rate
- `static` members exist outside any object
- `static` member function are not bound to any object, they do not have `this` pointer => As a result, 
  - `static` member functions may not be declared as `const`, 
  - and we may not refer to this in the body of a static member. 
  - This restriction applies both to explicit uses of this and to implicit uses of this by calling a nonstatic member

We can access a `static member` directly through the scope operator

```c++
double r;
r = Account::rate();
```

Even though `static members` are not part of the objects of its class, we can use an object, reference, or pointer of the class type to access a static member

```c++
Account ac1;
Account *ac2 = &ac1;
// equivalent ways to call the static member rate function
r=ac1.rate(); // through an Account object or reference 
r=ac2->rate(); // through a pointer to an Account object
```

`Member functions` can use static members directly, WITHOUT the scope operator

As with any other member function, we can define a `static member function ` inside or outside of the class body. 

- When we define a static member outside the class, we do not repeat the static keyword. 
- The keyword appears only with the declaration inside the class body

###### !! Need Review page. 300 - 304

