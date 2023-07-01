### Ch 8. The IO Library

Chapter 14 will look at how we can write our own input and output operators, and Chapter 17 will cover how to control formatting and how to perform random access on files

#### 8.1 The IO Classes

wide characters => use `w` , e.g. `wcin`, `wcout`, `wcerr`, `wifstream`

The library lets us ignore the differences among these different kinds of streams by using `inheritance`

- `ifstream` and `istringstream` inherit from `istream`

###### 8.1.1 No Copy or Assign for IO Objects

We cannot have a parameter or return type that is one of the stream types

- Functions that do IO typically pass and return the stream through references

```c++
ofstream out1, out2;
out1 = out2; // error: cannot assign stream objects 
ofstream print(ofstream);// error: can’t initialize the ofstream parameter 
out2 = print(out2); // error: cannot copy stream objects
```

###### 8.1.2 Condition States

Once an error has occurred, subsequent IO operations on that stream will fail. 

- We can read from or write to a stream ONLY when it is in a non-error state
- SO using a stream as a condition tells us only whether the stream is valid

Machine-dependent `iostate` => 4 possibilities see page. 313 Table 8.2

- The right way to determine the overall state of a stream is to use either good or fail

- The code that is executed when we use a stream as a condition is equivalent to calling `!fail()`

###### 8.1.3 Managing the Output Buffer

Using a buffer allows the operating system to combine several output operations from our program into a single system-level write. Because writing to a device can be time-consuming, letting the operating system combine several output operations into a single write can provide an important performance boost

- Output buffers are not flushed if the program terminates abnormally

There are several conditions that cause the buffer to be flushed

- The program completes normally. All output buffers are flushed as part of the return from main

- Sometimes buffer can become full, it will be flushed before writing the next value

- Explicitly using a manipulator, e.g.  `endl`, `flush`, `ends`

  `ends` inserts a null character into the buffer and then flushes it

- Use the `unitbuf` manipulator to set the stream’s internal state to empty the buffer after each output operation. By default, `unitbuf` is set for `cerr` (flushed immediately)

  ```c++
  cout << unitbuf; // all writes will be flushed immediately 
  cout << nounitbuf; // returns to normal buffering
  ```

- An output stream might be tied to another stream. In this case, the output stream is flushed whenever the stream to which it is tied is read or written

  - By default, `cin` and `cerr` are both tied to `cout`

- - Hence, reading `cin` or writing to `cerr` flushes the buffer in `cout`
  - Usage1: Takes no argument and returns a pointer to the output stream, if any, to which this object is currently tied
  - Usage 2: `x.tie(&o)` ties the stream x to the output stream o

#### 8.2 File Input and Output

In § 17.5.3 (p. 763) we’ll describe how to use the same file for both input and output

`fstream`-specific operations => page. 316 Table 8.3

###### 8.2.1 Using File Stream Objects

When we create a file stream, we can (optionally) provide a file name. When we supply a file name, open is called automatically

```c++
ifstream in(ifile);
```

When an `fstream` object is destroyed, `close` is called automatically

######  8.2.2 File Modes

6 modes => see page. 319 Table 8.4

- 6 restriction (detail for lookup)

File mode Is determined each time open is called



#### 8.3 `string` Streams

`stringstream`-Specific Operations => page. 321 Table 8.5

###### 8.3.1 Using an `istringstream`

An `istringstream` is often used when we have some work to do on an entire line, and other work to do with individual words within a line

###### 8.3.2 Using `ostringstream`

An `ostringstream` is useful when we need to build up our output a little at a time but do not want to print the output until later

