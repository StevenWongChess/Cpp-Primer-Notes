###### Notations in the book

1. Must-read => read book ikon

2. Skim => book ikon

3. Tricky but important => zoom ikon

4. Old important => italic bold

5. New => bold

864 pages to read in total, estimate time 20 days.



### Ch 1. Getting Started

#### 1.2 A First Look at Input/Output

share same window

- `cout`
- `cerr` - (standard error), for warning & error message
- `clog` - for general info

`cout << a << b` in essence is `(cout << a) << b`

`endl` automatically flush buffer

- Purpose: to avoid output stuck in buffer when program crash



#### 1.3 A Word about Comment

Good comment style

```c++
/*
 * Simple main function:
 * Read two numbers and write their sum 
 */
```

Comment Pairs  `/* */` Do Not Nest



#### 1.4 Flow of Control

###### 1.4.3 Reading an Unknown Number of Inputs

How? - use `while`

An `istream` becomes invalid when 

- we hit `end-of-file` (`ctrl-d`)  OR
- encounter an invalid input

3 common errors compiler can detect:

-  `syntax error`
-  `type error`
-  `declaration error`

