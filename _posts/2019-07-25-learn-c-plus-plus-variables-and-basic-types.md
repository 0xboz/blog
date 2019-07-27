---
title: "Learn C++: Variables and Basic Types"
date: 2019-07-26
tags: [C++]
excerpt: "C++ variables and basic types"
header:
    image: /assets/images/altitude_clouds_cold.jpg
    caption: "Photo credit: [**Adrien Olichon**](https://adrienolichon.fr/)"
---

*I have always been a fan of [C](https://en.wikipedia.org/wiki/C_(programming_language))/[C++](https://isocpp.org/) language family. Due to my education background, programming was never a focal point back then when I was in school. After a few hours of research on studying materials, I believe it is a great idea to document my journey of learning C++ as a beginner. Hopefully, this series will be helpful to those who are interested in learning C/C++ in the near future. This series is largely based on* __C++ Primer (5th Edition)__ *by [Stanley B. Lippman](https://en.wikipedia.org/wiki/Stanley_B._Lippman). It might be considered as some sort of study notes with a few tweaks from my personal preference.* 

### Learn C++: Table of Contents
[Getting Started](https://0xboz.github.io/blog/learn-c-plus-plus-getting-started/)  
Variables and Basic Types (*current post* - updating...)

<div class="notice--info">
  <p>Exercise 2.1: What are the differences between int, long, long long, and short? Between an unsigned and a signed type? Between a float and a double?</p>
</div>

```cpp
// int, long, long long, and short represent the integer values of different sizes, namely the number of bits in them.
// A signed type represents negative or positive numbers (including zero); an unsigned type represents only values greater than or equal to zero.
// The main difference between a float and a double lies in the precision. Typically, floats are represented in one word (32 bits) and doubles in two words (64 bits). In addition, the float and double types yield about 7 and 16 significant digits, respectively.
```
<div class="notice--info">
  <p>Exercise 2.2: To calculate a mortgage payment, what types would you use for the rate, principal, and payment? Explain why you selected each type.</p>
</div>

```cpp
// Rate: float
// Principal: long
// Payment: long
// The rate is usually a floating-point number with 4 significant digits. The principal and payment are integral usually less than 1 billion.
```
<div class="notice--info">
  <p>Exercise 2.3: What output will the following code produce?</p>
</div>

```cpp
unsigned u = 10, u2 = 42;
std::cout << u2 - u << std::endl;  // 32
std::cout << u - u2 << std::endl;  // Depends on the machine; 4294967264 = 2^32 − 32
int i = 10, i2 = 42;
std::cout << i2 - i << std::endl;  // 32
std::cout << i - i2 << std::endl;  // -32
std::cout << i - u << std::endl;  //  0
std::cout << u - i << std::endl;  //  0
```
<div class="notice--info">
  <p>Exercise 2.4: Write a program to check whether your predictions were correct. If not, study this section until you understand what the problem is.</p>
</div>

```cpp
#include <iostream>

int main() 
{
  unsigned u = 10, u2 = 42;
  std::cout << u2 - u << std::endl;  
  std::cout << u - u2 << std::endl;  

  int i = 10, i2 = 42;
  std::cout << i2 - i << std::endl; 
  std::cout << i - i2 << std::endl; 
  std::cout << i - u << std::endl;  
  std::cout << u - i << std::endl;  

  return 0;
}
```

<div class="notice--info">
  <p>Exercise 2.15: Which of the following definitions, if any, are invalid? Why?</p>
</div>

```cpp
// (a)
int ival = 1.01;  // ok: using a float to initialize int is likely to lose data. The fractional part will be truncated.
// (b)
int &rval1 = 1.01;  // error: initializer must be an int object
// (c)
int &rval2 = ival;  // ok
// (d)
int &rval3;  // error: a reference must be initialized when defined
```

<div class="notice--info">
  <p>Exercise 2.16: Which, if any, of the following assignments are invalid? If they are valid, explain what they do.</p>
</div>

```cpp
int i = 0, &r1 = i; double d = 0, &r2 = d;
// (a)
r2 = 3.14159;  // d = 3.14159
// (b)
r2 = r1;  // ok: d = i
// (c)
i = r2;  // ok: i = d
// (d)
r1 = d;  // ok: i = d
```

<div class="notice--info">
  <p>Exercise 2.17: What does the following code print?</p>
</div>

```cpp
int i, &ri = i;
i = 5; ri = 10;
std::cout << i << " " << ri << std::endl;  // 10 10
```

<div class="notice--info">
  <p>Exercise 2.18: Write code to change the value of a pointer. Write code to change the value to which the pointer points.</p>
</div>

```cpp
int main()
{
  int val = 1; 
  int *p = nullptr;  // initialize a null pointer
  p = &val;  // change the value of pointer p to the address of variable val
  *p = 2;   // change the variable val value to which the pointer p points
  return 0;
}
```

<div class='notice--info'>
  <p>Exercise 2.19: Explain the key differences between pointers and references.</p>
</div>

```cpp
// 1. (Regular) References must be initialized at declaration; pointers don't need to.
// 2. References are immutable during their life time.
// 3. References are not objects but alias of other objects; pointers are objects themselves holding the addresses of other objects.
```

<div class='notice--info'>
  <p>Exercise 2.20: What does the following program do?</p>
</div>

```cpp
int i = 42;  // define int i and initialize i with a value of 42 
int *p1 = &i;  // define p1 as a pointer to int, which points to the address of int i
*p1 = *p1 * *p1;  // change the value of i to which p1 points to; the new value of i equals to the previous i value squared
```
<div class='notice--info'>
  <p>Exercise 2.21: Explain each of the following definitions. Indicate whether any are illegal and, if so, why.</p>
</div>

```cpp
int i = 0;
// (a)
double* dp = &i;  // error: dp as a pointer to double, must point to a double; type mismatch
// (b)
int *ip = i;  // error: ip as a pointer to int, must point to the address of an int i, not int i itself
// (c)
int *p = &i;  // ok: p as a pointer to int points to the address of int i
```

<div class='notice--info'>
  <p>Exercise 2.22: Assuming p is a pointer to int, explain the following code:</p>
</div>

```cpp
if (p) // ...  // if p is not a null pointer, the statement is true; otherwise false
if (*p) // ... // if the int to which p points to is non-zero, the statement is true; otherwise false
```

<div class='notice--info'>
  <p>Exercise 2.23: Given a pointer p, can you determine whether p points to a valid object? If so, how? If not, why not?</p>
</div>

```cpp
// No, you can't. Because it would be expensive to maintain meta data about what constitutes a valid pointer and what doesn't, and in C++ you don't pay for what you don't want.
// https://stackoverflow.com/questions/17202570/c-is-it-possible-to-determine-whether-a-pointer-points-to-a-valid-object/17202622#17202622
```

<div class='notice--info'>
  <p>Exercise 2.24: Why is the initialization of p legal but that of lp illegal?</p>
</div>

```cpp
int i = 42;    
void *p = &i;  // ok: void* is a special pointer which can point to objects with a variety of types.
long *lp = &i;  // error: the pointer to long lp must hold the address of a long; type mismatch
```

<div class='notice--info'>
  <p>Exercise 2.25: Determine the types and values of each of the following variables.</p>
</div>

```cpp
// (a) 
int* ip, &r = ip;  // ip is a pointer to int; r is a reference to pointer ip
// (b) 
int i, *ip = 0;  // define int i; ip is a reference to int and initialized to null
// (c) 
int* ip, ip2;  // ip is a pointer to int; ip2 is an int
```

<div class="notice--info">
  <p>Exercise 2.26: Which of the following are legal? For those that are illegal,explain why.</p>
</div>

```cpp
// (a) 
const int buf;  // error: buf must be initialized after creation
// (b) 
int cnt = 0;  // ok
// (c) 
const int sz = cnt; // ok
// (d) 
++cnt, ++sz;  // ok for ++cnt; error for ++sz: const sz variable can NOT be changed after creation
```

> ...we can bind a reference to ```const``` to a ```nonconst``` object, a literal, or a more general expression...

>  A Reference to ```const``` May Refer to an Object That Is Not ```const```

Similarly,

> ...we can use a pointer to ```const``` to point to a ```nonconst`` object...

> A Point to ```const``` May Refer to an Object That Is Not ```const```

```const``` pointers won't allow us to change the address after its initialization.

> ...a pointer is itself const says nothing about whether we can use the pointer to change the underlying object...

That means we can change the value of the object of interest if the object itself is ```nonconst```, *vice versa*. 

<div class="notice--info">
  <p>Exercise 2.27: Which of the following initializations are legal? Explain why.</p>
</div>

```cpp
// (a) 
int i = -1, &r = 0;  // error: a regular reference may only be bound to an object, Not to a literal or to the result of a more general expression
// (b) 
int *const p2 = &i2;  // ok: a const pointer p2 may point to an int i2
// (c) 
const int i = -1, &r = 0;  // ok: on the contrary, a reference to const may be bound to a nonconst object, a literal, or a more general expression
// (d) 
const int *const p3 = &i2;  // ok: a const pointer p3 may point to an int i2 that may or may not be a const
// (e) 
const int *p1 = &i2;  // ok: a pointer p1 may point to an int i2 that may or may not be a const
// (f) 
const int &const r2;  // error: a const must be initialized; 
// (g) 
const int i2 = i, &r = i;  // ok: defined an int i2 and initialized at compile time; a reference to const r may be bound to an int i that may or may not be a const 
```

<div class="notice--info">
  <p>Exercise 2.28: Explain the following definitions. Identify any that are illegal.</p>
</div>

```cpp
// (a)
int i, *const cp;  // error: a const pointer must be initialized
// (b)
int *p1, *const p2;  // error: a const pointer must be initialized
// (c)
const int ic, &r = ic;  // error: a const must be initialized
// (d)
const int *const p3;  // error: a const pointer must be initialized
// (e)
const int *p;  // ok: legal but not recommended, since uninitialized pointers are a common source of run-time errors
```

<div class="notice--info">
  <p>Exercise 2.29: Using the variables in the previous exercise, which of the
following assignments are legal? Explain why.</p>
</div>

```cpp
// (a)
i = ic;  // error  
// (b)
p1 = p3;  // error 
// (c)
p1 = &ic;  // error 
// (d)
p3 = &ic;  // error 
// (e)
p2 = p1;  // error 
// (f)
ic = *p3;  // error 
```

<div class="notice--info">
  <p>Exercise 2.30: For each of the following declarations indicate whether the object being declared has top-level or low-level const.</p>
</div>

```cpp
const int v2 = 0;  // ok: high-level const v2
int v1 = v2;  // ok: ignore high-level const v2
int *p1 = &v1, &r1 = v1;  // ok
const int *p2 = &v2, *const p3 = &i, &r2 = v2;  // low-level const p2; p3 has both low-level and top-level const; low-level const r2, since all reference to const is low-level
```

<div class="notice--info">
  <p>Exercise 2.31: Given the declarations in the previous exercise determine whether the following assignments are legal. Explain how the top-level or low-level const applies in each case.</p>
</div>

```cpp
r1 = v2;  // ok: ignore high-level v2
p1 = p2;  // error: p2 has low-level const; low-level const doesn't match
p2 = p1;  // ok:
p1 = p3;  // error: p3 has low-level const; low-level const doesn't match 
p2 = p3;  // ok
```
<div class="notice--info">
  <p>Exercise 2.32: Is the following code legal or not? If not, how might you make it legal?</p>
</div>

```cpp
int main()
{
    // int null = 0, *p = null;  // error: invalid conversion from ‘int’ to ‘int*’, type mismatch

    // How to make it legal
    int null = 0, *p = &null;
    return 0;
}
```

All exercises and solutions can be found on [my github](https://github.com/0xboz/learn_c_plus_plus).

Stay tuned by signing up for [my newsletter](http://eepurl.com/gxmy39). If you have any questions/comments/proposals, feel free to shoot me a message on [Twitter](https://twitter.com/0xboz)/[Discord](https://discord.gg/jchMcc2)/[Patreon](https://www.patreon.com/0xboz). Happy coding!