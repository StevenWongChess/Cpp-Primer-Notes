### Ch 10. Generic Algorithms

#### 10.1 Overview

Most of the algorithms are defined in the `algorithm` header

The generic algorithms do not themselves execute container operations

- they have no way themselves to change the size of a container
- 1 Exception => algorithm on `inserter` might add elements



#### 10.2 A First Look at the Algorithms

More than 100 algorithm => check Appendix A

With only a few exceptions, the algorithms operate over a range of elements. We’ll refer to this range as the “input range”

###### 10.2.1 Read-Only Algorithms

```c++
string sum = accumulate(v.cbegin(), v.cend(), string(""));
// error:no + on const char*
string sum = accumulate(v.cbegin(), v.cend(), "");
```

 Algorithms that take a single iterator denoting a second sequence *assume* that the second sequence is at least as large at the first

- e.g. `equal`

###### 10.2.2 Algorithms That Write Container Elements

Algorithms do not check write operations

- e.g. `fill_n(dest, n, val)`

  ```c++
  vector<int>vec; // emptyvector
  // disaster: attempts to write to ten (nonexistent) elements in vec
  fill_n(vec.begin(), 10, 0);
  ```

When we assign through an `insert iterator`, a new element equal to the right-hand value is added to the container

`back_inserter` takes a reference to a container and returns an insert iterator bound to that container

- When we assign through that iterator, the assignment calls `push_back` to add an element with the given value to the container

```c++
vector<int>vec;// empty vector 
auto it = back_inserter(vec); // assigning through it adds elements to vec 
*it = 42; // vec now has one element with value 42
```

Mix with `fill_n`

```c++
vector<int> vec;// empty vector
// ok: back_inserter creates an insert iterator that adds elements to vec 
fill_n(back_inserter(vec), 10, 0); // appends ten elements to vec
```

`replace` vs its copy version `replace_copy`

```c++
replace(ilst.begin(), ilst.end(), 0, 42);
replace_copy(ilst.cbegin(), ilst.cend(), back_inserter(ivec), 0, 42);
```

###### 10.2.3 Algorithms That Reorder Container Elements

Eliminate adjacent duplicate

```c++
void elimDups(vector<string> &words) { 
  	sort(words.begin(), words.end());
		// unique reorders the input range so that each word appears once in the front portion of the range and returns an iterator one past the unique range 
  	auto end_unique = unique(words.begin(), words.end()); 
		words.erase(end_unique, words.end()); 
}
```



#### 10.3 Customizing Operations

###### 10.3.1 Passing a Function to an Algorithm

A `predicate` is an expression that can be called and that returns a value that can be used as a condition

- `e.g.`  first define `cmp` then `sort(v.begin(), v.end(), cmp);`

A stable sort maintains the original order among equal elements

- `stable_sort`



###### 10.3.2 Lambda Expressions

The predicates we pass to an algorithm must have exactly 1 or 2 parameters

- If more than 2 => Use lambda

3 kinds of `callable object`: if e is a callable expression, we can write e(args) where args is a comma-separated list of zero or more arguments. 

- functions and function pointers
- classes that overload the function- call operator, which we’ll cover in § 14.8 (p. 571)
- Lambda expression

```c++
[capture list](parameter list) -> return type {function body}
```

Lambda Expression:

- `capture list` is an (often empty) list of local variables defined in the enclosing function

- Unlike ordinary functions, a lambda may not have default arguments

- A lambda specifies the variables it will use by including those local non-static variables in its capture list (by value or by reference)

  ```c++
  void biggies(vector<string> &words, vector<string>::size_type sz,
  ostream &os = cout, char c = ’ ’)
  {
  		// code to reorder words as before
  		// statement to print count revised  toprint to os 
    	for_each(words.begin(), words.end(), [&os, c](const string &s){os << s << c });
  }
  ```

- The capture list is used for local nonstatic variables only; lambdas can use local statics and variables declared outside the function directly.



###### 10.3.3 Lambda Captures and Returns

When we define a lambda, the compiler generates a new (unnamed) class type that corresponds to that lambda. We’ll see how these classes are generated in § 14.8.1 (p. 572)

By default, the class generated from a lambda contains a data member corre- sponding to the variables captured by the lambda

Capture by Value:

- Because the value is copied when the lambda is created, subsequent changes to a captured variable have no effect on the corresponding value inside the lambda

```c++
void fcn1() {
  size_t v1 = 42; // local variable
  // copiesv1intothecallableobjectnamedf
  auto f = [v1] { return v1; };
  v1 = 0;
  auto j = f(); 
  // j is 42;fstored a copy ofv1when we created it
}
```

When we capture a variable by reference, we must ensure that the variable exists at the time that the lambda executes

- We can avoid potential problems with captures by minimizing the data we capture. Moreover, if possible, avoid capturing pointers or references

We can let the compiler infer which variables we use from the code in the lambda’s body.

- `[&]` or `[=]` or `[&, var1]` or `[=, &var1]`
- When we mix implicit and explicit captures, the first item in the capture list must be an `&` or `=`
- When we mix implicit and explicit captures, the explicitly captured variables must use the alternate form

Lambda Capture List => Table 10.1

Another way is instead of `[&]`, use `mutable` to change the value

```c++
void fcn3() {
		size_t v1 = 42; // local variable
		// f can change the value of the variables it captures
		auto f = [v1] () mutable { return ++v1; }; 
  	v1 = 0;
		auto j = f(); // j is 43
}
```

When we need to define a return type for a lambda, we must use a trailing return type

```c++
// ok
transform(vi.begin(), vi.end(), vi.begin(), [](int i) { return i < 0 ? -i : i; });
// error: cannot deduce the return type for the lambda
transform(vi.begin(), vi.end(), vi.begin(),
[](int i) { if (i < 0) return -i; else return i; });
// ok
transform(vi.begin(), vi.end(), vi.begin(), [](int i) -> int
{ if (i < 0) return -i; else return i; });
```

###### 10.3.4 Binding Arguments

If we need to do the same operation in many places, we should usually define a function rather than writing the same lambda expression multiple times. Similarly, if an operation requires many statements, it is ordinarily better to use a function.

it is not so easy to write a function to replace a lambda that captures local variables

- In order to use `check_size` in place of that lambda, we have to figure out how to pass an argument to the `sz` parameter

- ```c++
  bool check_size(const string &s, string::size_type sz) {
    return s.size() >= sz;
  }
  ```

- Solution: `bind` can be thought of as a general-purpose function adaptor

- ```c++
  auto newCallable = bind(callable, arg_list);
  ```
  
- ```c++
  // check6 is a callable object that takes one argument of type string 
  // and calls check_size on its given string and the value6 
  auto check6 = bind(check_size, _1, 6);
  ```

How to work with placeholders?

- `using std::placeholders::_1;` => define `placeholder` one by one, to make life easier => use `using namespace std::placeholders;`

- ```c++
  auto g = bind(f, a, b, _2, c, _1);
  g(_1, _2) // means
  f(a, b, _2, c, _1)
  ```

- because bind copies its arguments and we cannot copy an ostream. If we want to pass an object to bind without copying it, we must use the library `ref` OR `cref` function

- ```c++
  for_each(words.begin(), words.end(), bind(print, ref(os), _1, ’ ’));
  ```

- Modern C++ programs should use `bind`



#### 10.4 Revisiting Iterators

4 new iterators

- Insert iterator
- Stream iterator
- Reverse iterator
- Move iterator => see § 13.6.2 (p. 543)

###### 10.4.1 Insert Iterators

3 kinds of inserters => ops see Table 10.2

- `back_inserter` (§ 10.2.2, p. 382) creates an iterator that uses `push_back`
- `front_inserter` creates an iterator that uses `push_front`
- `inserter` creates an iterator that uses `insert`, insert ahead of second argument, which is another iterator

```c++
list<int> lst = {1,2,3,4};
list<int> lst2, lst3; // empty lists
// after copy completes, lst2 contains 4 3 2 1
copy(lst.cbegin(), lst.cend(), front_inserter(lst2));
// after copy completes, lst3 contains 1 2 3 4
copy(lst.cbegin(), lst.cend(), inserter(lst3, lst3.begin()));
```

###### 10.4.2 `iostream` Ierators

When we create a stream iterator, we must specify the type of objects that the iterator will read or write 

- See Table 10.3 for operations
- An iterator bound to a stream is equal to the end iterator once its associated stream hits end- of-file or encounters an IO error

```c++
istream_iterator<int>in_iter(cin); // read ints from cin 
istream_iterator<int> eof; // istream ‘‘end’’ iterator 
while (in_iter != eof) // while there’s valid input to read
  	vec.push_back(*in_iter++);

// another way
istream_iterator<int>in_iter(cin),eof; // read ints from cin 
vector<int>vec(in_iter, eof); // construct vec from an iterator range
```

We’ll see in § 10.5.1 (p. 410) how to tell which algorithms can be used with the stream iterators

- When we bind an `istream_iterator` to a stream, we are NOT guaranteed that it will read the stream immediately. The implementation is permitted to delay reading the stream until we use the iterator

```c++
istream_iterator<int> in(cin), eof; 
cout << accumulate(in, eof, 0) << endl;
```

Table 10.4 => see `ostream_iterator` operations

- We must bind an `ostream_iterator` to a specific stream. There is no empty or off-the-end `ostream_iterator`
- The `*` and `++` operators do nothing on an ostream_iterator, so omitting them has no effect on our program

```c++
ostream_iterator<int> out_iter(cout, " "); 
for (auto e : vec)
	*out_iter++ = e; // the assignment writes this element to cout 
cout << endl;
// equivalent but less clear way
for (auto e : vec)
	out_iter = e; // the assignment writes this element to cout
cout << endl;
```

###### 10.4.3 Reverse Iterator

Reverse iterator needs to support `++` and `--`

- `forward_list` does NOT have reverse iterators
- stream iterators do not, because it is not possible to move backward through a stream

See Figure 10.2 => Relation between reverse and ordinary iterators

- `cbegin()`
- `rcomma`
- `rcomma.base()`
- `crbegin()`
- `cend`



#### 10.5 Structure of Generic Algorithms

The iterator operations required by the algorithms are grouped into five iterator categories listed in Table 10.5

- Input iterator => Read, but not write; single-pass, increment only
- Output iterator => Write, but not read; single-pass, increment only
- Forward iterator => Read and write; multi-pass, increment only
- Bidirectional iterator => Read and write; multi-pass, increment and decrement
- Random-access iterator => Read and write; multi-pass, full iterator arithmetic

A second way is to classify the algorithms (as we did in the beginning of this chapter) is by whether they read, write, or reorder the elements in the sequence. Appendix A covers all the algorithms according to this classification

###### !!!10.5.1 The Five Iterator Categories

With the exception of output iterators, an iterator of a higher category provides all the operations of the iterators of a lower categories

NEED REVIEW page 411-412

###### 10.5.2 Algorithm Parameter Patterns

Most of the algorithms have one of the following four forms:

```c++
alg(beg, end, other args);
alg(beg, end, dest, other args); 
alg(beg, end, beg2, other args); // won't be checked
alg(beg, end, beg2, end2, other args);
```

Algorithms that write to an output iterator assume the destination is large enough to hold the output

###### 10.5.3 Algorithm Naming Conventions

some pass a predicate, e.g.

```c++
unique(beg, end); // uses the == operator to compare the elements 
unique(beg, end, comp); // uses comp to compare the elements
```

`_if` version

```c++
find(beg, end, val); // find the first instance of val in the input range 
find_if(beg, end, pred); // find the first instance for which pred is true
```

`_copy` version

```c++
reverse(beg, end); // reverse the elements in the input range 
reverse_copy(beg, end, dest);// copy elements in reverse order into dest
```

`_copy_if` version

```c++
// removes the odd elements from v1 
remove_if(v1.begin(), v1.end(), [](int i) { return i % 2; }); 
// copies only the even elements from v1 into v2; v1 is unchanged
remove_copy_if(v1.begin(), v1.end(), back_inserter(v2), [](int i) { return i % 2; });
```



#### 10.6 Container-Specific Algorithms

 The list member versions should be used in preference to the generic algorithms for `lists` and `forward_lists`

- See Table 10.6

`splice` for list => see Table 10.7

- NEED REVIEW

Most of the list-specific algorithms are similar—but not identical—to their generic counterparts. However, a crucially important difference between the list-specific and the generic versions is that the list versions change the underlying container
