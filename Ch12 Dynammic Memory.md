### Ch 12. Dynamic Memory

Properly freeing dynamic objects turns out to be a surprisingly rich source of bugs

#### 12.1 Dynamic Memory and Smart Pointers

###### 12.1.1 The `shared_ptr` Class

Table 12.1 (overleaf) lists operations common to `shared_ptr` and `unique_ptr`.

Those that are particular to `shared_ptr` are listed in Table 12.2 (p. 453)

The safest way to allocate and use dynamic memory is to call a library function named `make_shared`

When we copy or assign a `shared_ptr`, each `shared_ptr` keeps track of how many other `shared_ptrs` point to the same object

- It is up to the implementation whether to use a counter or another data structure to keep track of how many pointers share state

```c++
shared_ptr<int> p = make_shared<int>(42);
auto q(p);
```

The container classes are an example of classes that use dynamic memory for the first purpose and we’ll see examples of the second in Chapter 15

###### 12.1.2 Managing Memory Directly

We can prevent new from throwing an `bad_alloc` exception by using a different form of new

- For reasons we’ll explain in § 19.1.2 (p. 824) this form of new is referred to as placement new

```c++
int *p1 = new int;// if allocation fails, new throws std::bad_alloc
int *p2 = new (nothrow) int; // if allocation fails, new returns a null pointer
```

avoid `dangling pointer`

- `reset` (i.e. `p = nullptr`) after `delete` 
- In real systems, finding all the pointers that point to the same memory is surprisingly difficult

###### 12.1.3 Using `shared_ptrs` with `new`

The smart pointer constructors that take pointers are explicit (§ 7.5.4, p. 296)

- Hence, we cannot implicitly convert a built-in pointer to a smart pointer
- DO NOT mix `shared_ptr` with `new`
- It is dangerous to use a built-in pointer to access an object owned by a smart pointer, because we may not know when that object is destroyed

```c++
shared_ptr<int> p1 = new int(1024); // error: must use direct initialization 
shared_ptr<int> p2(new int(1024)); // ok: uses direct initialization
```

Other Ways to Define and Change `shared_ptrs` => Table 12.3

Although the compiler will not complain, it is an error to bind another smart pointer to the pointer returned by `get`

```c++
shared_ptr<int> p(new int(42)); // reference count is 1
int *q = p.get(); // ok: but don’t use q in any way that might delete its pointer 
{ // new block
// undefined: two independent shared_ptrs point to the same memory 
  shared_ptr<int>(q);
}// block ends,q is destroyed, and the memory to which q points is freed
int foo = *p; // undefined; the memory to which p points was freed
```

We can use reset to assign a new pointer to a `shared_ptr`

- The reset member is often used together with `unique` to control changes to the object shared among several `shared_ptrs`

```c++
p = new int(1024); // error: cannot assign a pointer to a shared_ptr 
p.reset(new int(1024)); // ok:p points to a new object
```

###### 12.1.4 Smart Pointers and Exceptions

When a function is exited, whether through normal processing or due to an exception, all the local objects are destroyed

- Including smart pointer

In contrast, memory that we manage directly is not automatically freed when an exception occurs

When we create a `shared_ptr`, we can pass an optional argument that points to a deleter function

```c++
void f(destination &d /* other parameters */) {
    connection c = connect(&d);
    shared_ptr<connection> p(&c, end_connection);
    // use the connection
    // when f exits, even if by an exception, the connection will be properly closed
}
```

###### 12.1.5 `unique_ptr`

Unlike `shared_ptr`, there is no library function comparable to `make_shared` that returns a `unique_ptr`. Instead, when we define a `unique_ptr`, we bind it to a pointer returned by `new`

- Because a `unique_ptr` owns the object to which it points, `unique_ptr` does not support ordinary copy or assignment

  ```c++
  unique_ptr<double> p1; // unique_ptr that can point at a double 
  unique_ptr<int> p2(new int(42)); // p2 points to int with value 42
  unique_ptr<string> p1(new string("Stegosaurus")); unique_ptr<string> p2(p1); // error: no copy for unique_ptr 
  unique_ptr<string> p3;
  p3 = p2; // error: no assign for unique_ptr
  ```

- Although we can’t copy or assign a unique_ptr, we can transfer ownership from one (nonconst) unique_ptr to another by calling release or reset

  ```c++
  // transfers ownership from p1(which points to the string Stegosaurus)to p2 
  unique_ptr<string> p2(p1.release()); // release makes p1 null
  unique_ptr<string> p3(new string("Trex"));
  // transfers ownership from p3 to p2
  p2.reset(p3.release()); // reset deletes the memory to which p2 had pointed
  ```

See operations => Table 12.4

1 exception: We can copy or assign a unique_ptr that is about to be destroyed

- The most common example is when we return a unique_ptr from a function

```c++
unique_ptr<int> clone(int p) {
// ok: explicitly create a unique_ptr<int> from int* 
  return unique_ptr<int>(new int(p));
}

unique_ptr<int> clone(int p) { 
  unique_ptr<int> ret(new int (p)); 
	// ...
	return ret;
}
```

However, for reasons we’ll describe in § 16.1.6 (p. 676), the way `unique_ptr` manages its deleter is differs from the way `shared_ptr` does.

```c++
unique_ptr<objT, delT> p (new objT, fcn);
```

###### 12.1.6 `weak_ptr`

Binding a `weak_ptr` to a `shared_ptr` does not change the reference count of that `shared_ptr`

- That object will be deleted even if there are `weak_ptrs` pointing to it
- The `lock` function checks whether the object to which the `weak_ptr` points still exists

See operations => Table 12.5



#### 12.2 Dynamic Arrays

For reasons we’ll explain in § 12.2.2 (p. 481), using an allocator generally provides better performance and more flexible memory management

###### 12.2.1 `new` and Arrays

 It is important to remember that what we call a dynamic array does not have an array type

- No `begin` and `end`
- Cannot use range for

Legal to `new char[0]`

```c++
char arr[0]; // error: cannot define a zero-length array 
char *cp = new char[0]; // ok: but cp can’t be dereferenced
```

The library provides a version of `unique_ptr` that can manage arrays allocated by `new` => see Table 12.6

```c++
unique_ptr<int[]> up(new int[10]);
up.release(); // automatically uses delete[] to destroy its pointer
```

Unlike `unique_ptr`, `shared_ptrs` provide no direct support for managing a dynamic array. If we want to use a `shared_ptr` to manage a dynamic array, we must provide our own deleter

###### 12.2.2 The `allocator` Class

In general, coupling allocation and construction can be wasteful

- `allocator` provides type-aware allocation of raw, unconstructed, memory
- In § 13.5 (p. 524), we’ll see an example of how this class is typically used
- Operations => see Table 12.7

Correct Order: Allocate => construct => destroy => deallocate

- Table 12.8 Easier construction



#### 12.3 Using the Library: A Text-Query Program

=> Need review the EXAMPLE (page. 484 - 490)





