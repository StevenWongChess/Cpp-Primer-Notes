### Ch 13. Copy Control

NEED to review 13.6 !!!

Copy control

- Copy constructor
- Copy assignment operator
- Move constructor
- Move assignment operator
- destructor

If a class does not define all of the copy-control members, the compiler automatically defines the missing operations

#### 13.1 Copy, Assign and Destroy

###### 13.1.1 The Copy Constructor

A constructor is the copy constructor if 

- its first parameter is a reference to the class type 
- and any additional parameters have default values

When we do not define a copy constructor for a class, the compiler synthesizes one for us

- Unlike the synthesized default constructor (§ 7.1.4, p. 262), a copy constructor is synthesized even if we define other constructors
- As we’ll see in § 13.1.6 (p. 508), the **synthesized copy constructor** for some classes prevents us from copying objects of that class type
- Otherwise, the synthesized copy constructor **memberwise copies** the members of its argument into the object being created

We are now in a position to fully understand the differences between 

- direct initialization 
- and copy initialization (§ 3.2.1, p. 84)

Copy initialization ordinarily uses the copy constructor. 

- However, as we’ll see in § 13.6.2 (p. 534), if a class has a move constructor, then copy initialization sometimes uses the move constructor instead of the copy constructor

Copy initialization happens not only when

- we define variables using an =, but also when we
- Pass an object as an argument to a parameter of non-reference type
  - explains why the copy constructor’s own parameter must be a reference
- Return an object from a function that has a nonreference return type
- Brace initialize the elements in an array or the members of an aggregate class (§ 7.5.5, p. 298)
- Some class types also use copy initialization for the objects they allocate
  - For example, the library containers copy initialize their elements when we initialize the container, or when we call an insert or push member (§ 9.3.1, p. 342)
  - By contrast, elements created by an emplace member are direct initialized (§ 9.3.1, p. 345).

As we’ve seen, whether we use copy or direct initialization matters if we use an initializer that requires conversion by an explicit constructor (§ 7.5.4, p. 296)

```c++
vector<int> v1(10); // ok: direct initialization
vector<int> v2 = 10; // error: constructor that takes a size is explicit
void f(vector<int>); // f’s parameter is copy initialized
f(10); // error: can’t use an explicit constructor to copy an argument 
f(vector<int>(10)); // ok: directly construct a temporary vector from an int
```

During copy initialization, the compiler is permitted (but not obligated) to skip the copy/move constructor and create the object directly

###### 13.1.2 The Copy-Assignment Operator

To be consistent with assignment for the built-in types (§ 4.4, p. 145), assignment operators usually return a reference to their left-hand operand

```c++
class Foo {
		public:
		Foo& operator=(const Foo&); // assignment operator 
  	// ...
};
```

Just as it does for the copy constructor, the compiler generates a synthesized copy- assignment operator for a class if the class does not define its own

###### 13.1.3 The `destructor`

Because it takes no parameters, it cannot be overloaded. There is always only one destructor for a given class

In a constructor, members are initialized before the function body is executed, and members are initialized in the same order as they appear in the class. 

- In a destructor, the function body is executed first and then the members are destroyed. Members are destroyed in reverse order from the order in which they were initialized

The destructor is used automatically whenever an object of its type is destroyed

- Variables are destroyed when they go out of scope
- Members of an object are destroyed when the object of which they are a part is destroyed
- Elements in a container—whether a library container or an array—are destroyed when the container is destroyed
- Dynamically allocated objects are destroyed when the delete operator is applied to a pointer to the object
- Temporary objects are destroyed at the end of the full expression in which the temporary was created

###### 13.1.4 The Rule of Three/Five

as we’ll see in § 13.6 (p. 531), under the new standard, a class can also define a move constructor and move-assignment operator

We can treat these 5 as a whole unit

If the class needs a destructor, it almost surely needs a copy constructor and copy-assignment operator as well

- Copy pointer (but pointed to same object) => delete twice (can't delete same object) error

If a class needs a copy constructor, it almost surely needs a copy-assignment operator. And vice versa

###### 13.1.5 Using `= default`

When we specify `= default` on the declaration of the member inside the class body

- the synthesized function is implicitly inline
- (just as is any other member function defined in the body of the class)
- If we do not want the synthesized member to be an inline function, we can specify `= default` on the member’s definition, as we do in the definition of the copy-assignment operator

###### 13.1.6 Preventing Copies

for some classes, there really is no sensible meaning for these operations

- For example, the iostream classes prevent copy- ing to avoid letting multiple objects write to or read from the same IO buffer
- Can not delete destructor

In essence, the copy-control members are synthesized as deleted when it is impossible to copy, assign, or destroy a member of the class

- => 4 cases in page. 508



#### 13.2 Copy Control and Resource Management

Ordinarily, classes that manage resources that do not reside in the class must define the copy-control members

###### 13.2.1 Classes That Act Like Values

Assignment operators typically combine the actions of the destructor and the copy constructor

-  even if an object is assigned to itself

```c++
HasPtr& HasPtr::operator=(const HasPtr &rhs) {
	auto newp = new string(*rhs.ps); // copy the underlying string
  delete ps; // free the old memory
  ps = newp; // copy data from rhs into this object
  i = rhs.i; 
  return *this; // return this object
}
```

###### 13.2.2 Defining Classes That Act Like Pointers

The easiest way to make a class act like a pointer is to use `shared_ptrs` to manage the resources in the class

- sometimes we want to manage a resource directly. In such cases, it can be useful to use a `reference count `



#### 13.3  Swap

Conceptually it’s easy to see that swapping two objects involves a copy and two assignments

 Unlike the copy-control members, swap is never necessary. However, defining swap can be an important optimization for classes that allocate resources

For reasons we’ll explain in § 16.3 (p. 697), if there is a type-specific version of swap, that version will be a better match than the one defined in std

- Very careful readers may wonder why the using declaration inside swap does not hide the declarations for the HasPtr version of swap (§ 6.4.1, p. 234). We’ll explain the reasons for why this code works in § 18.2.3 (p. 798)

Classes that define swap often use swap to define their assignment operator

 Assignment operators that use copy and swap are automatically excep- tion safe and correctly handle self-assignment



#### 13.4 A Copy-Control Example

page.519 to 524



#### 13.5 Classes That Manage Dynamic Memory

e.g. `StrVec` class page.524-531

`move`

- First, for reasons we’ll explain in § 13.6.1 (p. 532), when reallocate con- structs the strings in the new memory it must call move to signal that it wants to use the string move constructor. If it omits the call to move the string the copy constructor will be used
- Second, for reasons we’ll cover in § 18.2.3 (p. 798), we usually do not provide a using declaration (§ 3.1, p. 82) for move. When we use move, we call std::move, not move



#### 13.6 Moving Objects

In some of these circumstances, an object is immediately destroyed after it is copied

- In those cases, moving, rather than copying, the object can provide a significant performance boost
- A second reason to move rather than copy:  The IO and `unique_ptr `classes can be moved but not copied

###### 13.6.1 Rvalue References

We can bind an rvalue reference to these kinds of expressions, but we cannot directly bind an rvalue reference to an lvalue

```c++
int i = 42;
int &r = i;				// ok:r refers to i
int &&rr = i;			// error: cannot bind an rvalue reference to an lvalue
int &r2 = i * 42;	// error:i*42 is an rvalue
const int &r3 = i * 42;	// ok: we can bind a reference to const to an rvalue
int &&rr2 = i * 42;	 // ok: bind rr2 to the result of the multiplication
```

Although we cannot directly bind an rvalue reference to an lvalue

- we can explicitly cast an lvalue to its corresponding rvalue reference type
- Another way: The move function uses facilities that we’ll describe in § 16.2.6 (p. 690) to return an rvalue reference to its given object

```c++
int &&rr3 = std::move(rr1); // ok
```

 Code that uses move should use `std::move`, not `move`. Doing so avoids potential name collisions

- We’ll explain the reasons for this usage in § 18.2.3 (p. 798)

###### 13.6.2 Move Constructor and Move Assignment

NEVER free lhs before using sources from rhs =>NO exception allowed

- `move constructor` example
- We must specify noexcept on both the declaration in the class header and on the definition if that definition appears outside the class
- more detail in § 18.1.4 (p. 779)

```c++
StrVec::StrVec(StrVec &&s) noexcept // move won’t throw any exceptions 
  	// member initializers take over the resources in s
		: elements(s.elements), first_free(s.first_free), cap(s.cap){
		// leave s in a state in which it is safe to run the destructor 
    s.elements = s.first_free = s.cap = nullptr;
}
```

To avoid this potential problem, vector must use a copy constructor instead of a move constructor during reallocation *unless it knows* that the element type’s move constructor cannot throw an exception => page 536

 After a move operation, the “moved-from” object must remain a valid, destructible object but users may make no assumptions about its value.

As it does for the copy constructor and copy-assignment operator, the compiler will synthesize the move constructor and move-assignment operator

- However, the conditions under which it synthesizes a move operation are quite different from those in which it synthesizes a copy operation
- Differently from the copy operations, for some classes the compiler does not synthesize the move operations *at all*
- The compiler will synthesize a move constructor or a move-assignment operator *only* if the class doesn’t define any of its own copy-control members and if every nonstatic data member of the class can be moved

With one important exception, the rules for when a synthesized move operation is defined as deleted are analogous to those for the copy operations (§ 13.1.6, p. 508) 

- => NEED REVIEW page 538-543 
- and `move iterator`

###### 13.6.3 Rvalue References and Member Functions

Member functions other than constructors and assignment can benefit from providing both copy and move versions











