###### Notations in the book

1. Must-read =><img src="/Users/stevenwong/Downloads/BE_SDE/VE280/c++_primer_image/1.png" style="zoom:50%;" />

2. Skim =><img src="/Users/stevenwong/Downloads/BE_SDE/VE280/c++_primer_image/2.png" style="zoom:50%;" />

3. Tricky =><img src="/Users/stevenwong/Downloads/BE_SDE/VE280/c++_primer_image/3.png" style="zoom:50%;" />

4. Old important => italic bold

5. New => bold

864 pages to read in total, estimate time 20 days.



### Ch 1. Getting Started

#### 1.2 A First Look at Input/Output

`cerr`(standard error), `clog`

`endl` automatically flush buffer



#### 1.3 A Word about Comment

Good comment style (can not nest)

```c++
/*
 * Simple main function:
 * Read two numbers and write their sum 
 */
```

Comment Pairs  `/* */` Do Not Nest



#### 1.4 Flow of Control

Reading an Unknown Number of Inputs => `while`

An `istream` becomes invalid when we hit 

- `end-of-file` (`ctrl-d`)  OR
- encounter an invalid input

3 Common compile errors:

-  `syntax error`
-  `type error`
-  `declaration error`

