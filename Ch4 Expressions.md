### Ch 4. Expressions

#### 4.1 Fundamentals

Chapter 14 will show how we can define operators for our own types

###### 4.1.1 Basic Concepts

Roughly speaking, when we use an object as an `rvalue`, we use the object’s value (its contents). When we use an object as an `lvalue`, we use the object’s identity (its location in memory)

The important point is that 

- with 1 exception that we’ll cover in § 13.6 page. 531
- we can use an lvalue when an rvalue is required, 
- BUT we cannot use an rvalue when an lvalue (i.e., a location) is required

Lvalues and rvalues also differ when used with `decltype`

- When we apply decltype to an expression (other than a variable), the result is a reference type if the expression yields an lvalue. As an example, assume p is an `int*`. Because dereference yields an lvalue, `decltype(*p)` is `int&`. On the other hand, because the address-of operator yields an rvalue, `decltype(&p)` is `int**`, that is, a pointer to a pointer to type int.

###### 4.1.3 Order of Evaluation

We have no way of knowing whether `f1` will be called before `f2` or vice versa

```c++
int i = f1() * f2();
```

For operators that do not specify evaluation order, it is an error for an expression to refer to and change the same object

```c++
int i = 0;
cout << i << " " << ++i << endl; // undefined
```

There are four operators that do guarantee the order in which operands are evaluated: `&&` `||` `?:` `,`



#### 4.4 Assignment Operators

The left-hand operand of an assignment operator must be a modifiable lvalue, the result of an assignment is its left-hand operand, which is an lvalue

Unlike the other binary operators, assignment is right associative

Assignment has low precedence, so parentheses are usually needed around assignments in conditions



#### 4.5 Increment and Decrement Operators

Use postfix only when necessary

- Prefix: increments its operand and return the object itself as an lvalue
- Postfix: increment the operand but yield a copy of the original, unchanged value as an rvalue



#### 4.6 The Member Access Operators

The arrow operator requires a pointer operand and yields an lvalue. The dot operator yields an lvalue if the object from which the member is fetched is an lvalue; otherwise the result is an rvalue.



#### 4.8 The Bitwise Operator

As we’ll see in § 17.2 (p. 723), we can also use these operators on a library type named `bitset` that represents a flexibly sized collection of bits

Use `unsigned` types for bitwise operators

- Because NO guarantees for how the sign bit is handled

The left-shift operator (the **<<** **operator**) inserts 0-valued bits on the right. 

The behavior of the right-shift operator (the **>>** **operator**) depends on the type of the left-hand operand: 

- If that operand is unsigned, then the operator inserts 0-valued bits on the left;
- If it is a signed type, the result is implementation defined—either copies of the sign bit or 0-valued bits are inserted on the left.

An overloaded operator has the same precedence and associativity as the built-in version of that operator



#### 4.9 The `sizeof` Operator

`sizeof (type)` 

`sizeof expr`, sizeof returns the size of the type returned by the given expression (`sizeof` does not evaluate its operand)

` sizeof` an array is the size of the entire array. It is equivalent to taking the sizeof the element type times the number of elements in the array. Note that sizeof does not convert the array to a pointer.

`sizeof` a string or a vector returns only the size of the fixed part of these types; it does not return the size used by the object’s elements



#### 4.10 Comma Operator

The comma operator `,` guarantees the order in which its operands are evaluated

The result of a comma expression is the value of its right-hand expression. The result is an lvalue if the right-hand operand is an lvalue

- e.g. used in for loop



#### 4.11 Type Conversions

The implicit conversions among the arithmetic types are defined to preserve precision, if possible

The compiler automatically converts operands (**Implicit Conversions**) in the following circumstances

- In most expressions, values of integral types smaller than int are first promoted to an appropriate larger integral type.
- In conditions, nonbool expressions are converted to bool.
- In initializations, the initializer is converted to the type of the variable; in assignments, the right-hand operand is converted to the type of the left-hand.
- In arithmetic and relational expressions with operands of mixed types, the types are converted to a common type.
- As we’ll see in Chapter 6, conversions also happen during function calls.

###### 4.11.1 Operands of Unsigned Type

Unsigned type

- If both (possibly promoted) operands have the same signedness, then the operand with the smaller type is converted to the larger type
- When the signedness differs and the type of the unsigned operand is the same as or larger than that of the signed operand, the signed operand is converted to unsigned
- The remaining case is when the signed operand has a larger type than the unsigned operand. In this case, the result is machine dependent.

###### 4.11.2 Other Implicit Conversions

In most expressions, when we use an array, the array is automatically converted to a pointer to the first element in that array. 

- This conversion is not performed when an array is used with decltype or as the operand of the address-of (&), sizeof, or typeid (which we’ll cover in § 19.2.2 (p. 826)) operators. 
- The conversion is also omitted when we initialize a reference to an array (§ 3.5.1, p. 114). 
- As we’ll see in § 6.7 (p. 247), a similar pointer conversion happens when we use a function type in an expression

There are several other pointer conversions:

- A constant integral value of 0 and the literal nullptr can be converted to any pointer type
- a pointer to any nonconst type can be converted to `void*`
- a pointer to any type can be converted to a `const void*`
- We’ll see in § 15.2.2 (p. 597) that there is an additional pointer conversion that applies to types related by inheritance
- We can convert a pointer to a `nonconst` type to a pointer to the corresponding const type, and similarly for references

Class types can define conversions that the compiler will apply automatically. 

- The compiler will apply ONLY one class-type conversion at a time. 
- In § 7.5.4 (p. 295) we’ll see an example of when multiple conversions might be required, and will be rejected

###### 4.11.3 Explicit Conversions

Explicit conversions: `cast-name<type>(expression);`

- The *cast-name* may be one of **static_cast**, **dynamic_cast**, **const_cast**, and **reinterpret_cast**. We’ll cover dynamic_cast, which supports the run-time type identification, in § 19.2 (p. 825)

- A `static_cast` is often useful when a larger arithmetic type is assigned to a smaller type

- A `static_cast` is also useful to perform a conversion that the compiler will not generate automatically. For example, we can use a static_cast to retrieve a pointer value that was stored in a void* pointer (§ 2.3.2, p. 56)

- A `const_cast` changes only a low-level (§ 2.4.3, p. 63) const in its operand

- A `const_cast` is most useful in the context of overloaded functions, which we’ll describe in § 6.4 (p. 232)

- A `reinterpret_cast` generally performs a low-level reinterpretation of the bit pattern of its operands

  ```c++
  int *ip;
  char *pc = reinterpret_cast<char*>(ip);
  ```

- A `reinterpret_cast` is inherently machine dependent. Safely using reinterpret_cast requires completely understanding the types involved as well as the details of how the compiler implements the cast

No longer use old style

```c++
BAD! type (expr); // function-style cast notation 
BAD! (type) expr; // C-language-style cast notation
```



#### 4.12 Operator Precedence Table

See page. 166-167 Table 4.4

