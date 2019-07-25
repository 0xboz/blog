---
title: "Learn C++: Variables and Basic Types"
date: 2019-07-18
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

All exercises and solutions can be found on [my github](https://github.com/0xboz/learn_c_plus_plus).

Stay tuned by signing up for [my newsletter](http://eepurl.com/gxmy39). If you have any questions/comments/proposals, feel free to shoot me a message on [Twitter](https://twitter.com/0xboz)/[Discord](https://discord.gg/jchMcc2)/[Patreon](https://www.patreon.com/0xboz). Happy coding!