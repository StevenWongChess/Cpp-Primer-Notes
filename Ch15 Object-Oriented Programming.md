### Ch15. Object-Oriented Programming

#### 15.1 OOP: An Overview

The key ideas in object-oriented programming are 

- data abstraction, 
- inheritance, 
- and dynamic binding (use objects of these types while ignoring the details of how they differ)

A derived class may include the virtual keyword on these functions but is not required to do so

 In C++, `dynamic binding (runtime binding)` happens when a virtual function is called through a reference (or a pointer) to a base class



#### 15.2 Defining Base and Derived Classes

###### 15.2.1 Defining a Base Class

 Base classes ordinarily should define a virtual destructor. 

- Virtual destructors are needed even if they do no work
- We’ll explain virtual destructors in § 15.7.1 (p. 622)

`virtual` function => dynamic (runtime) binding

- Member functions that are not declared as virtual are resolved at compile time, not run time

###### 15.2.2 Defining a Derived Class

Although the standard does not specify how derived objects are laid out in memory

- We can refer to Figure 15.1

the compiler will apply the derived-to-base conversion implicitly

a derived class must use a base-class constructor to initialize its base-class part

- The base class is initialized first, and then the members of the derived class are initialized in the order in which they are declared in the class

We’ll have more to say about scope in § 15.6 (p. 617), but for now it’s worth knowing that the scope of a derived class is nested inside the scope of its base class

we can use a static member through either the base or derived

A class must be defined, not just declared, before we can use it as a base class

- One implication of this rule is that it is impossible to derive a class from itself

Prevent inheritance => use `final`

```c++
class NoDerived final{/* */};// NoDerived can’t be a base class
```

###### 15.2.3 Conversions and Inheritance

We can bind a pointer or reference to a base-class type to an object of a type derived from that base class

- 1 crucially important implication: When we use a reference (or pointer) to a base-class type, we don’t know the actual type of the object to which the pointer or reference is bound

The dynamic type of an expression that is neither a reference nor a pointer is always the same as that expression’s static type

- The static type of an expression is always known at compile time
- The dynamic type may not be known until run time

When we initialize or assign an object of a base type from an object of a derived type, only the base-class part of the derived object is copied, moved, or assigned. 

- The derived part of the object is ignored



#### 15.3 Virtual Functions

We MUST define every virtual function, regardless of whether it is used, because the compiler has no way to determine whether a virtual function is used

The key idea behind OOP is polymorphism

- The fact that the static and dynamic types of references and pointers can differ is the cornerstone of how C++ supports polymorphism

To override virtual function => same argument types + same return type (one exception, § 15.5 p. 613 + example in § 15.8.1 p. 633)

As we’ll see in § 15.6 (p. 620), it is legal for a derived class to define a function with the same name as a virtual in its base class but with a different parameter list => a bug when people try to override but has a typo => hard to find => use `override`

Use scope operator `::` to use a particular version of that virtual

-  Ordinarily, only code inside member functions (or friends) should need to use the scope operator to circumvent the virtual mechanism

```c++
double undiscounted = baseP->Quote::net_price(42);
```



#### 15.4 Abstract Base Classes

Unlike ordinary virtuals, a pure virtual function does not have to be defined

We cannot (directly) create objects of a type that is an abstract base class

We can ONLY use `constructor` of its direct base class 



#### 15.5 Access Control and Inheritance

Just as friendship is NOT transitive (§ 7.3.4, p. 279), friendship is also NOT inherited

Sometimes we need to change the access level of a name that a derived class inherits

- We can do so by providing a `using` declaration (§ 3.1, p. 82)

By default, a derived class defined with the class keyword has private inheritance; a derived class defined with struct has public inheritance

```c++
classBase{/* ... */};
struct D1 : Base { /* . . . */ }; // public inheritance by default 
class D2 : Base { /* . . . */ }; // private inheritance by default
```



#### 15.6 Class Scope under Inheritance

Under inheritance, the scope of a derived class is nested (§ 2.2.4, p. 48) inside the scope of its base classes

- name defined in derived class will hide name defined in base class 
- use scope operator to access

Function name same => The base member is hidden even if the functions have different parameter lists

```c++
struct Base {
		int memfcn();
};
struct Derived : Base { 
  	int memfcn(int); // hides memfcn in the base
};
Derived d; Base b; 
b.memfcn();// calls Base::memfcn
d.memfcn(10); // calls Derived::memfcn
d.memfcn(); // error: memfcn with no arguments is hidden 
d.Base::memfcn()// ok: calls Base::memfcn
```

If the base and derived members took arguments that differed from one another, there would be no way to call the derived version through a reference or pointer to the base class. For example => MUST same parameter in base & derived for virtual functons (needs review)

```c++
class Base {
public:
    virtual int fcn();
};
class D1 : public Base {
public:
		// hides fcn in the base; this fcn is not virtual
		// D1 inherits the definition of Base::fcn()
		int fcn(int); // parameter list differs from fcn in Base 
  	virtual void f2(); // new virtual function that does not exist in Base
};
class D2 : public D1 {
public:
		int fcn(int);// nonvirtual function hides D1::fcn(int) 
  	int fcn(); // overrides virtual fcn from Base
		void f2(); // overrides virtual f2 from D1
};
Base bobj; D1 d1obj; D2 d2obj;
Base *bp1 = &bobj, *bp2 = &d1obj, *bp3 = &d2obj; 
bp1->fcn();// virtual call, will call Base::fcnatruntime 
bp2->fcn();// virtual call, will call Base::fcnatruntime 
bp3->fcn();// virtual call, will call D2::fcnatruntime

D1 *d1p = &d1obj; D2 *d2p = &d2obj; 
bp2->f2();// error:Base has no member named f2 
d1p->f2();// virtual call, will call D1::f2() at runtime 
d2p->f2();// virtual call, will call D2::f2() at runtime

Base *p1 = &d2obj; D1 *p2 = &d2obj; D2 *p3 = &d2obj; 
p1->fcn(42); // error: Base has no version of fcn that takes an int 
p2->fcn(42); // statically bound, calls D1::fcn(int) 
p3->fcn(42); // statically bound, calls D2::fcn(int)
```

If a derived class wants to make all the overloaded versions available through its type, then it must override all of them or none of them => what if want to override some => `using`



#### 15.7 Constructors and Copy Control

###### 15.7.1 Virtual Destructors

Destructors for base classes are an important exception to the rule of thumb that if a class needs a destructor, it also needs copy and assignment (§ 13.1.4, p. 504)

-  If a class defines a destructor— even if it uses = default to use the synthesized version—the compiler will not synthesize a move operation for that class (§ 13.6.2, p. 537)

###### 15.7.2 Synthesized Copy Control and Inheritance



OLD NOTES

1. Virtual destructor => turn off synthesized `move` (see 15.7.2)
2. `delete` is inherited
3. Unlike the constructors and assignment operators, the destructor is responsible only for destroying the resources allocated by the derived class
4. for copy move we MUST explicitly use the copy (or move, assignment) constructor for the base class in the derived’s constructor initializer list
5. If a constructor or destructor calls a virtual, the version that is run is the one corresponding to the type of the constructor or destructor itself
6. A derived class inherits its base-class constructors by providing a using dec- laration that names its (direct) base class
7. Unlike using declarations for ordinary members, a constructor using declaration does not change the access level of the inherited constructor(s) + a using declaration can’t specify `explicit` or `constexpr` + default not inherited 
8. When we need a container that holds objects related by inheritance, we typically define the container to hold pointers (preferably smart pointers (§ 12.1, p. 450)) to the base class
9. => irony: have no choice but to use pointers instead of object!!!
10. Skipped 15.8, 15.9 (example in pp. 634 to 649)







