### Ch 14. Overloaded Operations and Conventions

#### 14.1 Basic Concepts

Except for the overloaded function-call operator, operator(), an overloaded operator may not have default arguments (§ 6.5.1, p. 236).

Cannot override operands of built-in type

We’ll cover overloading new and delete in § 19.1.1 (p. 820)

An overloaded operator has the same precedence and associativity (§ 4.1.2, p. 136) as the corresponding built-in operator. Regardless of the operand types

```c++
data1 + data2; // normal expression 
operator+(data1, data2); // equivalent function call
data1 += data2; // expression-based ‘‘call’’ 
data1.operator+=(data2); // equivalent call to a member operator function
```

Recall that a few operators guarantee the order in which operands are evaluated. Because using an overloaded operator is really a function call, these guarantees do not apply to overloaded operators

- the operand-evaluation guarantees of the logical AND, logical OR, and comma operators are not preserved
- overloaded versions of && or || operators do not preserve short-circuit evaluation properties of the built-in operators. Both operands are always evaluated
- Ordinarily, the comma, address-of, logical AND, and logical OR opera- tors should *not* be overloaded.



#### 14.2 Input and Output Operators

###### 14.2.1 Overloading the Output Operator `<<`

Generally, `<<` should do minimal formatting

`<<` MUST be nonmember functions

###### 14.2.2 Overloading the Input Operator `>>`

N/A



#### 14.3 Arithmetic and Relational Operators

###### 14.3.1 Equality Operators

If a class defines operator==, it should also define operator!=

###### 14.3.2 Relational Operators

N/A



#### 14.4 Assignment Operators

 Assignment operators can be overloaded. Assignment operators, regardless of parameter type, must be defined as member functions



#### 14.5 Subscript Operator

The subscript operator must be a member function

If a class has a subscript operator, it usually should define two versions: one that returns a plain reference and the other that is a const member and returns a reference to const



#### 14.6 Increment and Decrement Operators

Classes that define increment or decrement operators should define both the prefix and postfix versions. These operators usually should be defined as members

 To be consistent with the built-in operators, the prefix operators should return a reference to the incremented or decremented object

There is one problem with defining both the prefix and postfix operators: Normal overloading cannot distinguish between these operators

- To solve this problem, the postfix versions take an extra (unused) parameter of type int

Calling the Postfix Operators Explicitly

```c++
p.operator++(0); // call postfix operator++ 
p.operator++(); // call prefix operator++
```



#### 14.7 Member Access Operators

Operator arrow must be a member

 The overloaded arrow operator *must* return either a pointer to a class type or an object of a class type that defines its own operator arrow



#### 14.8 Function-Call Operator

The function-call operator must be a member function

###### 14.8.1 Lambdas Are Function Objects

When we write a lambda, the compiler translates that expression into an unnamed object of an unnamed class (§ 10.3.3, p. 392)

- Essence of lambda => lambdas Are Function Objects

```c++
// get an iterator to the first element whose size() is >= sz 
auto wc = find_if(words.begin(), words.end(), [sz](const string &a)
                  
// would generate a class that looks something like
class SizeComp {
	SizeComp(size_t n): sz(n) { } // parameter for each captured variable 
// call operator with the same return type, parameters, and body as the lambda
	bool operator()(const string &s) const{ 
    return s.size() >= sz; 
  } 
private:
	size_t sz; // a data member for each variable captured by value 
};
```

Classes generated from a lambda expression have a deleted default constructor, deleted assignment operators, and a default destructor

- Whether the class has a defaulted or deleted copy/move constructor depends in the usual ways on the types of the captured data members

###### 14.8.2 Library-Defined Function Objects

Library Function Objects => Table 14.2

The function-object classes that represent operators are often used to override the default operator used by an algorithm

###### 14.8.3 Callable Objects and `function`

`Callable function` = 

- functions and pointers to functions
- lambdas (§ 10.3.2, p. 388)
- objects created by bind (§ 10.3.4, p. 397)
- and classes that overload the function-call operator

two callable objects with different types may share the same **call signature**

```c++
int(int, int)
```

- Old solution: use map to define a **function table**
- C++11 => `function`

```c++
function<int(int, int)> f1 = add; // function pointer
function<int(int, int)> f2 = divide(); // object of a function-object class
function<int(int, int)> f3 = [](int i, int j){ return i * j; };// lambda

cout << f1(4,2) << endl; // prints 6 
cout << f2(4,2) << endl; // prints 2 
cout << f3(4,2) << endl; // prints 8
```



#### 14.9 Overloading, Conversions, and Operators

=> NEED REVIEW

###### 14.9.1 Conversion Operators

A **conversion operator** is a special kind of member function that converts a value of a class type to a value of some other type

- A conversion function typically has the general form

  `operator type() const;`

- must be defined as member functions

```c++
int i = 42;
cin << i; // this code would be legal if the conversion to bool were not explicit!
```

to prevent => `explicit conversion operators`

- As with an explicit constructor (§ 7.5.4, p. 296), the compiler won’t (generally) use an explicit conversion operator for implicit conversions
- 4 Exception => the compiler will still apply an explicit conversion to an expression used as a condition
  - `if`, `while`, `do`
  - `for`
  - `!` `||` `&&`
  - `? :`

###### 14.9.2 Avoiding Ambiguous Conversions

If there is more than one way to perform a conversion, it will be hard to write unambiguous code

- There are two ways that multiple conversion paths can occur
  - when two classes provide mutual conversions
  - define multiple conversions from or to types that are themselves related by conversions
- Note that we can’t resolve the ambiguity by using a cast—the cast itself would have the same ambiguity, must explicitly call
- see example in page.584

###### 14.9.3 Function Matching and Overloaded Operators

 The set of candidate functions for an operator used in an expression can contain both nonmember and member functions



