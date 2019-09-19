---
title: "How to Run and Debug C++ with Visual Studio Code on Debian"
date: 2019-08-07
tags: [C++, Visual Studio Code, Debian]
excerpt: "Run and debug C++ with Visual Studio Code on Debian"
header:
    image: /assets/images/beautiful_motion.jpg
    caption: "Photo credit: [**John Cahil Rom**](https://www.pexels.com/@cahilrom)"
---

[Visual Studio Code](https://code.visualstudio.com/) is a popular source code editor among many developers. This tutorial is going to show you how to run and debug lightweight C++ codes in it. 

Since my desktop runs on Debian, you might need to make a few minor changes for Windows PC and macOS. 

### Install Compiler ```g++```

```
sudo apt install build-essential
```

### Run C++ Code
This can be done by installing [Code Runner](https://marketplace.visualstudio.com/items?itemName=formulahendry.code-runner) extension.

<figure>
    <a href="{{ site.url }}{{ site.baseurl }}/assets/images/code_runner.png">
        <img src="{{ site.url }}{{ site.baseurl }}/assets/images/code_runner.png">
    </a>
    <figcaption>Code Runner extension</figcaption>
</figure>

If your program needs to take user input, we can modify a few things in VSCode user settings.  

Initiate Command Palette by ```Ctrl + Shift + P``` and search ```Open User Settings```. If you would like to make this change globally, click the curly bracket on the right corner. In case you prefer this change only within the current workspace, select ```Workspace``` tab first and then click the curly bracket.

<figure>
    <a href="{{ site.url }}{{ site.baseurl }}/assets/images/vscode_user_settings.png">
        <img src="{{ site.url }}{{ site.baseurl }}/assets/images/vscode_user_settings.png">
    </a>
    <figcaption>Code Runner extension</figcaption>
</figure>

Add the following line.

```json
"code-runner.runInTerminal": true,
```

Furthermore, we can expand the settings to support newer functions introduced in C++14 and C++17 by appending one more line.

```json
"code-runner.executorMap": {
    "cpp": "cd $dir && g++ -std=c++14 $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt"
},
```

You should be able to compile your ```.cpp``` files and generate executable programs now. 

### Debug C++ Code

Here comes another question: how can I utilize VSCode debug option with C++ codes?

First of all, let us check if our system has GNU Debugger installed.

```
sudo apt install gdb
```

We also need to install another [VSCode extension C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools).

<figure>
    <a href="{{ site.url }}{{ site.baseurl }}/assets/images/microsoft_c_cplusplus_extension.png">
        <img src="{{ site.url }}{{ site.baseurl }}/assets/images/microsoft_c_cplusplus_extension.png">
    </a>
    <figcaption>Microsoft C/C++ extension</figcaption>
</figure>

Now, we can open a ```.cpp``` file and hit ```F5```.

Select ```Select C++ (GDB/LLDB)```

<figure>
    <a href="{{ site.url }}{{ site.baseurl }}/assets/images/vscode_cpp_debug_1.png">
        <img src="{{ site.url }}{{ site.baseurl }}/assets/images/vscode_cpp_debug_1.png">
    </a>    
</figure>

Select ```g++ build and debug active file```.

<figure>
    <a href="{{ site.url }}{{ site.baseurl }}/assets/images/vscode_cpp_debug_2.png">
        <img src="{{ site.url }}{{ site.baseurl }}/assets/images/vscode_cpp_debug_2.png">
    </a>    
</figure>

Two ```json``` files will be created in directory ```.vscode``` - ```launch.json``` and ```tasks.json```. There is no need to change any default settings in those two files. You should be ready to debug C++ source codes by ```F5``` now.

I hope you liked this short tutorial. Stay tuned by signing up for [my newsletter](http://eepurl.com/gxmy39). If you have any questions/comments/proposals, feel free to shoot me a message on [Twitter](https://twitter.com/0xboz)/[Discord](https://discord.gg/JHt7UQu)/[Patreon](https://www.patreon.com/0xboz). 

Happy coding!