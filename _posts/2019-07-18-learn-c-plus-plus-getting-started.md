---
title: "Learn C++: Getting Started"
date: 2019-07-18
tags: [C++]
excerpt: "Get started with C++"
header:
    image: /assets/images/mac_book_black_and_white.jpg
    caption: "Photo credit: [**Negative Space**](https://negativespace.co/)"
---

*I have always been a fan of [C](https://en.wikipedia.org/wiki/C_(programming_language))/[C++](https://isocpp.org/) language family. Due to my education background, programming was never a focal point back then when I was in school. After a few hours of research on studying materials, I believe it is a great idea to document my journey of learning C++ as a beginner. Hopefully, this series will be helpful to those who are interested in learning C/C++ in the near future. This series is largely based on* __C++ Primer (5th Edition)__ *by [Stanley B. Lippman](https://en.wikipedia.org/wiki/Stanley_B._Lippman). It might be considered as some sort of study notes with a few tweaks from my personal preference.* 

### Learn C++: Table of Contents
Getting Started (*current post*)  
[Variables and Basic Types](https://0xboz.github.io/blog/learn-c-plus-plus-variables-and-basic-types/)

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

> Exercise 1.3: Write a program to print Hello, World on the standard output.

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

>  Exercise 1.4: Our program used the addition operator, +, to add two numbers. Write a program that uses the multiplication operator, *, to print the product instead.

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

>  Exercise 1.5: We wrote the output in one large statement. Rewrite the program to use a separate statement to print each operand.

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
> Exercise 1.6: Explain whether the following program fragment is legal.

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

## Flow of Control

### ```while``` Statement

```
    while (condition)
        statement
```
Let us take a look at the solutions of some exercises. 

> Exercise 1.9: Write a program that uses a while to sum the numbers from 50 to 100.

```cpp
/*
 * Exercise 1.9: Write a program that uses a while to sum the numbers from 50 to 100.
 */
#include <iostream>

int main()
{
    int val = 50, sum = 0;
    while (val <= 100) {
        sum += val;
        ++val;
    }
    std::cout << "The sum from 50 to 100 is " << sum << std::endl;
    return 0;
}
```
> Exercise 1.10: In addition to the ++ operator that adds 1 to its operand, there is a decrement operator (--) that subtracts 1. Use the decrement operator to write a while that prints the numbers from ten down to zero.

```cpp
/*
 * Exercise 1.10: In addition to the ++ operator that adds 1 to its operand,
 * there is a decrement operator (--) that subtracts 1. Use the decrement
 * operator to write a while that prints the numbers from ten down to zero.
 */
#include <iostream>

int main()
{
    int val = 10;
    while (val >= 0) {
        std::cout << "Count down " << val << std::endl;
        --val;
    };
    return 0;
}
```

> Exercise 1.11: Write a program that prompts the user for two integers. Print each number in the range specified by those two integers.

```cpp
/*
 * Exercise 1.11: Write a program that prompts the user for two integers.
 * Print each number in the range specified by those two integers.
 */
#include <iostream>

int main()
{
    std::cout << "Please enter two numbers: " << std::endl;
    int val1 = 0, val2 = 0;
    std::cin >> val1 >> val2;

    // Assume we do not know "if" statement
    while (val1 < val2) {
        std::cout << val1 << std::endl;
        ++val1;
    };
    while (val2 < val1) {
        std::cout << val2 << std::endl;
        ++val2;
    };
    while (val1 == val2) {
        std::cout << val2 << std::endl;
        ++val1;
    };

    return 0;
}
```

### ```for``` Statement

> Exercise 1.16: Write your own version of a program that prints the sum of a set of integers read from cin.

```cpp
/*
 * Exercise 1.16: Write your own version of a program that prints the sum of
 * a set of integers read from cin.
 */
#include <iostream>

int main()
{
    std::cout << "Please enter a set of integers. Press Ctrl + D when you are done." << std::endl;
    int sum = 0;
    // Reading an Unknown Number of Inputs
    for (int val = 0; std::cin >> val; sum += val) {
    }
    std::cout << "\nThe sum is " << sum << std::endl;
    return 0;
}
```

### ```if``` Statement

> Exercise 1.19: Revise the program you wrote for the exercises in § 1.4.1 (p. 13) that printed a range of numbers so that it handles input in which the first number is smaller than the second.

```cpp
/*
 * Exercise 1.19: Revise the program you wrote for the exercises in § 1.4.1 (p.
 * 13) that printed a range of numbers so that it handles input in which the first
 * number is smaller than the second.
 *
 * Here is the original excercise.
 * Exercise 1.11: Write a program that prompts the user for two integers.
 * Print each number in the range specified by those two integers.
 *
 */
#include <iostream>

int main()
{
    std::cout << "Please enter two numbers: " << std::endl;
    int val1 = 0, val2 = 0;
    std::cin >> val1 >> val2;

    // Assume we do not know "if" statement
    // while (val1 < val2) {
    //     std::cout << val1 << std::endl;
    //     ++val1;
    // };
    // while (val2 < val1) {
    //     std::cout << val2 << std::endl;
    //     ++val2;
    // };
    // while (val1 == val2) {
    //     std::cout << val2 << std::endl;
    //     ++val1;
    // };

    // Implement "if" statement
    if (val1 <= val2) {
        while (val1 <= val2) {
            std::cout << val1 << std::endl;
            ++val1;
        }
    } else {
        while (val2 <= val1) {
            std::cout << val2 << std::endl;
            ++val2;
        }
    }
    return 0;
}
```

## Classes

> Exercise 1.20: http://www.informit.com/title/032174113 contains a copy of Sales_item.h in the Chapter 1 code directory. Copy that file to your working directory. Use it to write a program that reads a set of book sales transactions, writing each transaction to the standard output.

For your convenience, you can also download a copy from [my github](https://raw.githubusercontent.com/0xboz/learn_c_plus_plus/master/Sales_item.h). We have two options to solve this problem: ```while``` statement and ```for``` statement. Here is the code.

```cpp
/*
 * Exercise 1.20: http://www.informit.com/title/032174113 contains a copy of
 * Sales_item.h in the Chapter 1 code directory. Copy that file to your
 * working directory. Use it to write a program that reads a set of book sales
 * transactions, writing each transaction to the standard output.
 */
#include <iostream>
#include "Sales_item.h"

//  Solution with "while" statement
int main()
{
    Sales_item book;
    while(std::cin >> book){
        std::cout << book << std::endl;
    }
    return 0;
}

// Alternative solution with "for" statement
// int main()
// {
//     for (Sales_item book; std::cin >> book; std::cout << book << std::endl) {

//     }
//     return 0;
// }
```

In addition, we can use file redirection to save tedious terminal typing.

```
$ YourCompiledProgram <infile >outfile
```
<div class="notice--info">
  <p>Interestingly, the angle bracket ```<``` in file redirection is somehow opposite to ```cin``` and ```cout```. </p>
</div>

Here is my input file.
```
0-201-78345-X 3 20.00
0-201-78145-X 2 25.00
0-201-78845-X 7 16.00
0-201-70353-X 4 24.99
```
And the output file is as follows.
```
0-201-78345-X 3 60 20
0-201-78145-X 2 50 25
0-201-78845-X 7 112 16
0-201-70353-X 4 99.96 24.99
```

> Exercise 1.21: Write a program that reads two Sales_item objects that have the same ISBN and produces their sum.

```cpp
/*
 * Exercise 1.21: Write a program that reads two Sales_item objects that
 * have the same ISBN and produces their sum.
 */
#include <iostream>
#include "Sales_item.h"

int main()
{
    std::cout << "Enter two items with the same ISBN: " << std::endl;
    Sales_item item1, item2;
    std::cin >> item1 >> item2;
    std::cout << item1 + item2 << std::endl;
    return 0;
}
```

> Exercise 1.22: Write a program that reads several transactions for the same ISBN . Write the sum of all the transactions that were read.

```cpp
/*
 * Exercise 1.22: Write a program that reads several transactions for the same
 * ISBN . Write the sum of all the transactions that were read.
 */
#include <iostream>
#include "Sales_item.h"

int main()
{
    std::cout << "Enter all transactions with the same ISBN: " << std::endl;
    Sales_item total, entry;
    while(std::cin >> entry){
        total += entry;
    };
    std::cout << total << std::endl;
    return 0;
}
```

> Exercise 1.23: Write a program that reads several transactions and counts how many transactions occur for each ISBN.
> Exercise 1.24: Test the previous program by giving multiple transactions representing multiple ISBN s. The records for each ISBN should be grouped together.

```cpp
/*
 * Exercise 1.23: Write a program that reads several transactions and counts
 * how many transactions occur for each ISBN.
 *
 * Exercise 1.24: Test the previous program by giving multiple transactions
 * representing multiple ISBN s. The records for each ISBN should be grouped
 * together.
 */
#include <iostream>
#include "Sales_item.h"

int main()
{
    Sales_item total;
    if (std::cin >> total) {
        Sales_item trans;
        while (std::cin >> trans) {
            if (total.isbn() == trans.isbn()) {
                total += trans;
            } else {
                std::cout << total << std::endl;
                total = trans;
            }
        }
        std::cout << total << std::endl;
    }
    return 0;
}
```

By now, we should be ready for a deep dive into C++ basics. All exercises and solutions are also available on [my github](https://github.com/0xboz/learn_c_plus_plus).

Stay tuned by signing up for [my newsletter](http://eepurl.com/gxmy39). If you have any questions/comments/proposals, feel free to shoot me a message on [Twitter](https://twitter.com/0xboz)/[Discord](https://discord.gg/jchMcc2)/[Patreon](https://www.patreon.com/0xboz). Happy coding!