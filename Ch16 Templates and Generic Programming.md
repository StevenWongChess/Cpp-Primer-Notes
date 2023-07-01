### Ch 16. Templates and Generic Programming

(HARD!!! needs review), OLD NOTES

1. `template <typename T, class U> calc (const T&, const U&);`

2. `nontype parameter`  => An argument bound to a `nontype` integral parameter must be a constant expression. Arguments bound to a pointer or reference `nontype` parameter must have static lifetime

   ```c++
   template<unsigned N, unsigned M>
   int compare(const char (&p1)[N], const char (&p2)[M]) {
   		return strcmp(p1, p2);
   }
   compare("hi", "mom");
   ```

3. unlike nontemplate code, headers for templates typically include definitions as well as declarations

4. Find error => errors can be detected during instantiation (usually link time)

5. EXAMPLE => template version of StrBlob (§ 12.1.1, p. 456) => pp.659 to 668

6. By default, the language assumes that a name accessed through the scope op- erator is not a type => use `typename`

7. **default template arguments**

   ```c++
   template <typename T, typename F = less<T>>
   int compare(const T &v1, const T &v2, F f = F()) {
   		if (f(v1, v2)) return -1;
   		if (f(v2, v1)) return 1;
   		return 0;  
   }
   ```