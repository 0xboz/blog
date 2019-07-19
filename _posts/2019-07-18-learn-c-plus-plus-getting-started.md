---
title: "Learn C++: Getting Started"
date: 2019-07-18
tags: [C++]
excerpt: "Get started with C++"
header:
    image: /assets/images/mac_book_black_and_white.jpg
    caption: "Photo credit: [**Negative Space**](https://negativespace.co/)"
---

I have always been a fan of [C](https://en.wikipedia.org/wiki/C_(programming_language))/[C++](https://isocpp.org/) language family. Due to my education background, programming was never a huge focus back then when I was in school. After a few hours of research on studying materials, I believe it is a great idea to document my journey of learning C++ as a beginner. Hopefully, this series will be helpful to those non-tech savy folks who are curious about C/C++. This series is largely based on *C++ Primer (5th Edition)* by [Stanley B. Lippman](https://en.wikipedia.org/wiki/Stanley_B._Lippman). It might be considered as some sort of study notes with a few tweaks from my personal setup. 

## Compiler
Since my OS is [Debian](https://www.debian.org/), installing the common compiler is quite straight-forward. You might need to make some changes accordingly if you are using [Microsoft Windows](https://en.wikipedia.org/wiki/Microsoft_Windows) or [MacOS](https://en.wikipedia.org/wiki/MacOS). 

```
$ sudo apt update && sudo apt install build-essential
```
Verify g++ installation
```
$ g++ -v
``` 
Here is mine.
```
gcc version 6.3.0 20170516 (Debian 6.3.0-18+deb9u1) 
```

## Text Editor
I am planning to go on with [Sublime Text](https://www.sublimetext.com/) for a while, and might switch to [Visual Studio Code](https://code.visualstudio.com/) later on. To be honest, you will do just fine with any text editor at this moment.

## A Simple C++ Program
Let us create a file named ```ex1.1.cpp```. There are a few common suffix conventions like ```.c```, ```.cp```, ```.cpp```, and ```.cc```. It really depends on the local compiler of your choice. Here is the simple program.

```cpp
int main()
{
    return 0;
}
```
Open the terminal in the same directory where ```ex1.1.cpp``` is located.
```
$ g++ -o ex1.1 ex1.1.cpp
```
A new file is generated named ```ex1.1```. In the same terminal, run the following.
```
$ ./ex1.1
```
You might scratch your head and ask why it shows nothing. Do not worry, it is a good sign that you have got nothing. Wanna see the results? Execute this command in the terminal.
```
$ echo $?
```
> A return value of 0 indicates success. Ordinarily a nonzero return indicates what kind of error occurred

What if we change ```return 0``` to ```return -1```? 

<div class="notice--info">
  <p>This question is cited from Lippman's book.</p>
  <p>Exercise 1.2: Change the program to return -1. A return value of -1 is often treated as an indicator that the program failed. Recompile and rerunyour program to see how your system treats a failure indicator from main.</p>
</div>

In my case, it returns ```255```.

## Hello, World!
How could we go on without this most influential one?! Interestingly enough, C++ language itself does not take care of statements for input or output (IO). An extensive standard library is introduced to manage this task (and many others as well). Here is our code.

```cpp
/* 
 * Exercise 1.3: Write a program to print Hello, World on the standard output. 
 */
#include <iostream>

int main()
{
    std::cout << "Hello, World!\n" << std::endl;
    return 0;
}
```
<div class="notice--warning">
  <p>The last letter of ```endl``` is ```l```, other than the number ```1```. At least, that was a mistake I have made while reading the book.</p>
</div>

## More Exercises
The best way to learn a programming language is to code. Here are some solutions to the exercises from the book.

```cpp
/*
 * Exercise 1.4: Our program used the addition operator, +, to add two
 * numbers. Write a program that uses the multiplication operator, *, to print
 * the product instead.
 */
#include <iostream>

int main()
{
    std::cout << "Please enter two numbers: " << std::endl;
    int v1 = 0, v2 = 0;
    std::cin >> v1 >> v2;
    std::cout << "The multiplication of " << v1 << " and " << v2 << " is " << v1 * v2 << std::endl;
    return 0;
}
```

```cpp
/*
 * Exercise 1.5: We wrote the output in one large statement. Rewrite the
 * program to use a separate statement to print each operand.
 */
#include <iostream>

int main()
{
    std::cout << "Please enter two numbers: " << std::endl;
    int v1 = 0, v2 = 0;
    std::cin >> v1 >> v2;
    std::cout << "The multiplication of ";
    std::cout << v1;
    std::cout << " and ";
    std::cout << v2;
    std::cout << " is ";
    std::cout << v1 * v2;
    std::cout << std::endl;
    return 0;
}
```

```cpp
/*
 * Exercise 1.6: Explain whether the following program fragment is legal.
 */
#include <iostream>

int main()
{
    std::cout << "Please enter two numbers: " << std::endl;
    int v1 = 0, v2 = 0;
    std::cin >> v1 >> v2;
    std::cout << "The multiplication of " << v1;
        << " and " << v2;
        << " is " << v1 * v2 << std::endl;
    return 0;
}
```
As to Excercise 1.6, the answer is *not legal*.
```
ex1.6.cpp:9:9: error: expected primary-expression before ‘<<’ token
```

## Updating...