### Ch 5. Statements

#### 5.1 Simple Statements

 Null statements should be commented. That way anyone reading the code can see that the statement was omitted intentionally

- An empty block is equivalent to a null statement



#### 5.2 Statement Scope

Variables defined in the control structure are visible only within that statement and are out of scope after the statement ends

- `if`, `switch`, `while`, and `for`



#### 5.3 Conditional Statements

###### 5.3.1 `if` Statement

`Dangling else` : In C++ the ambiguity is resolved by specifying that each else is matched with the closest preceding unmatched if

###### 5.3.2 `switch` Statement

Switch

- After a case label is matched, execution starts at that label and continues across all the remaining cases or until the program explicitly interrupts it
- two case labels same value => error
- `case` wrong type or non-constant => error
- How to define variable in `case`



#### 5.4 Iterative Statements

###### 5.4.1 `while` Statement

Variables defined in a while condition or while body are created and destroyed on each iteration

###### 5.4.3 Range `for` Statement

Now that we know how a range for works, we can understand why we said in § 3.3.2 (p. 101) that we cannot use a range for to add elements to a vector (or other container)

- In a range `for`, the value of `end()` is cached
- If we add elements to (or remove them from) the sequence, the value of end might be invalidated
- We’ll have more to say about these matters in § 9.3.6 (p. 353)



#### 5.5 Jump Statements

C++ offers four jumps: break, continue, and goto, which we cover in this chapter, and the return statement, which we’ll describe in § 6.3 (p. 222)

###### 5.5.3 `goto` Statement

 Programs should not use `gotos`. `gotos` make programs hard to understand and hard to modify.

```c++
goto label;
lable:
	xxx
  xxx
```



#### 5.6 `try` Blocks and Exception Handling

We’ll also have more to say about exceptions in § 18.1 (p. 772)

###### 5.6.2 `try` Block

`err.what()`

- Each of the library exception classes defines a member function named `what`. These functions take no arguments and return a C-style character string (i.e., a `const char*`)

The search for a handler reverses the call chain

- If no appropriate catch is found, execution is transferred to a library function named **terminate**. The behavior of that function is system dependent but is guaranteed to stop further execution of the program

Writing exception safe code is surprisingly HARD, and (largely) beyond the scope of this language Primer.

- Programs that do handle exceptions and continue processing generally must be constantly aware of whether an exception might occur and what the program must do to ensure that objects are valid, that resources don’t leak, and that the program is restored to an appropriate state.

###### 5.6.3 Standard Exceptions

4 headers for standard exceptions:

- The `exception` header defines the most general kind of exception class named exception. It communicates only that an exception occurred but provides no additional information.
- The `stdexcept` header defines several general-purpose exception classes, which are listed in => page.198, Table 5.1.
- The `new` header defines the `bad_alloc` exception type, which we cover in § 12.1.2 (p. 458).
- The `type_info` header defines the `bad_cast` exception type, which we cover in § 19.2 (p. 825).
- We can only default initialize (§ 2.2.1, p. 43) `exception`, `bad_alloc`, and `bad_cast` objects; it is not possible to provide an initializer for objects of these exception types.
- The other exception types have the **opposite** behaviour: We can initialize those objects from either a string or a C-style string, but we cannot default initialize them
- The exception types define only a single operation named `what`. That function takes no arguments and returns a `const char*` that points to a C-style character string (§ 3.5.4, p. 122). The purpose of this C-style character string is to provide some sort of textual description of the exception thrown.

