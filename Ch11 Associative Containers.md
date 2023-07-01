### Ch 11. Associative Containers

9 associative containers: 2 X 2 X 2

- `Set` or `map`
- Unique or multiple key
- In order or not

#### 11.1 Using an Associative Container

NA



#### 11.2 Overview of the Associative Containers

Associative containers (both ordered and unordered)

- Support the general container operations covered in § 9.2 (p. 328) and listed in Table 9.2 (p. 330)
- do NOT support the sequential-container position-specific operations, such as `push_front`
- do not support the constructors or insert operations that take an element value and a count
- provide some operations (Table 11.7 (p. 438)) and type aliases (Table 11.3 (p. 429)) that the sequential containers do not
- iterators are bidirectional (§ 10.5.1, p. 410)

###### 11.2.1 Define an Associative Container

The keys in a map or a set must be unique; there can be only one element with a given key. The multimap and multiset containers have no such restriction

###### 11.2.2 Requirements on Key Type

We’ll cover the requirements for keys in the unordered containers in § 11.4 (p. 445)

For the ordered containers (`map`, `multimap`, `set`, and `multiset`)

- the key type must define a way to compare the elements
- By default, the library uses the < operator for the key type to compare the keys

We can define compare function for our own type, remembering that when we use decltype to form a function pointer, we must add a `*` to indicate that we’re using a pointer to the given function type `(§ 6.7, p. 250)`

```c++
multiset<own_type, decltype(cmp)*> bookstore(cmp);
```

###### 11.2.3 The `pair` Type

See => Page 427 Table 11.2

Unlike other library types, the data members of pair are public



#### 11.3 Operations on Associative Containers

In addition to the types listed in Table 9.2 (p. 330), the associative containers define the types listed in Table 11.3

###### 11.3.1 Associative Container Iterators

Although the set types define both the iterator and const_iterator types, both types of iterators give us read-only access to the elements in the set

- Just as we cannot change the key part of a map element, the keys in a set are also const

In general, we do not use the generic algorithms (Chapter 10) with the associative containers. 

- The fact that the keys are const means that we cannot pass associative container iterators to algorithms that write to or reorder container elements

###### 11.3.2 Adding Elements

As we’ve seen, under the new standard the easiest way to create a pair is to use brace initialization inside the argument list

See Table 11.4 for Associative Container `insert` Operations

`insert` return what?

- For the containers that have unique keys, 

  The first member of the pair is an iterator to the element with the given key; the second is a bool indicating whether that element was inserted, or was already there

  If the key is already in the container, then insert does nothing, and the bool portion of the return value is false. If the key isn’t present, then the element is inserted and the bool is true

- For the containers that allow multiple keys

  the insert operation that takes a single element returns an iterator to the new element. There is no need to return a bool, because insert always adds a new element in these types

###### 11.3.3 Erasing Elements

See => Table 11.5

###### 11.3.4 Subscripting a `map`

Subscripting a map behaves quite differently from subscripting an array or vector: 

- Using a key that is not already present *adds* an element with that key to the map
- Ordinarily, the type returned by dereferencing an iterator and the type returned by the subscript operator are the same. Not so for maps: when we subscript a map, we get a mapped_type object; when we derefer- ence a map iterator, we get a value_type object

###### 11.3.5 Accessing Elements

See => Table 11.7 

- `c.lower_bound(k)` Returns an iterator to the first element with key not less than k
- `c.upper_bound(k)` Returns an iterator to the first element with key greater than k

Thus, calling lower_bound and upper_bound on the same key yields an iterator range (§ 9.2.1, p. 331) that denotes all the elements with that key

- If there is no element for this key, then lower_bound and upper_bound will be equal
- we can use `equal_range` instead page.440



#### 11.4 The Unordered Containers

The new standard defines four unordered associative containers. 

- Rather than using a comparison operation to organize their elements, 
- these containers use a hash function and the key type’s `==` operator

Although hashing gives better average case performance in principle, achieving good results in practice often requires a fair bit of performance testing and tweaking. 

- As a result, it is usually easier (and often yields better performance) to use an ordered container
- the performance of an unordered container depends on the quality of its hash function and on the number and size of its buckets

Aside from operations that manage the hashing, the unordered containers provide the same operations (find, insert, and so on) as the ordered containers

How to manage the buckets?

- Table 11.8

However, we cannot directly define an unordered container that uses a our own class types for its key type. Unlike the containers, we cannot use the hash template directly. Instead, we must supply our own version of the hash template. We’ll see how to do so in § 16.5 (p. 709).

```c++
size_t hasher(const Sales_data &sd) {
    return hash<string>()(sd.isbn());
}
bool eqOp(const Sales_data &lhs, const Sales_data &rhs) {
    return lhs.isbn() == rhs.isbn();
}
using SD_multiset = unordered_multiset<Sales_data, decltype(hasher)*, decltype(eqOp)*>;
SD_multiset bookstore(42, hasher, eqOp);
```







