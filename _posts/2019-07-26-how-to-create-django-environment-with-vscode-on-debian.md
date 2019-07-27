---
title: "How to Create Django Environment With Visual Studio Code on Debian"
date: 2019-07-26
tags: [Django, Visual Studio Code, Debian]
excerpt: "Create Django environment with VS Code on Debian"
header:
    image: /assets/images/beautiful_blossom.jpg
    caption: "Photo credit: [**Pixabay**](https://pixabay.com/)"
---

[Django](https://www.djangoproject.com/) is a popular high-level Python framework for web development. In this tutorial, we are going to do a quick walk-through of setting up Django development environment with [Visual Studio Code](https://code.visualstudio.com/) on Debian.

## Visual Studio Code
There are [two major ways](https://code.visualstudio.com/docs/setup/linux#_debian-and-ubuntu-based-distributions)(```.deb``` package and repo) to install VS Code in linux distros based on Debian. Since I usually don't use ```.deb``` or add any other repo into my Debian OS for security reasons, we will go through another route - [snap store](https://snapcraft.io/store).

#### Install ```snap``` on Debian
```
sudo apt install snapd
```
#### Install VS Code via Snap Store
```
sudo snap install --classic code
```

## Django
As always, it is highly recommended to use a virtual environment to manage the packages as the project grows.

#### Create ```venv```
```
mkdir django_project  #  Create a directory for our demo project
cd django_project
python3 -m venv venv
```

#### Launch VS Code
In the same terminal, we are going to launch VS Code. For some reason, I need to append ```--disable--gpu``` in my setup.

```
code . --disable-gpu
```
The next step is to install Python extension for VS Code. This can be done through its GUI.

<figure>
    <a href="{{ site.url }}{{ site.baseurl }}/assets/images/python_extension_vscode.png">
        <img src="{{ site.url }}{{ site.baseurl }}/assets/images/python_extension_vscode.png">
    </a>
    <figcaption>VS Code Python extension</figcaption>
</figure>

Further, we need to select Python interpreter for our project. Hit ```Ctrl+Shift+P``` and search ```Python: Select Interpreter```. From the list, select the directory that starts with ```./venv```

#### Install ```django```
We can create a new integrated terminal in VS Code by ``` Ctrl+` ```. By default, the venv we created earlier will be activated automatically. 
```
(venv) pip install -U pip  # Update pip
(venv) pip install django
```

That's it! I hope you liked this short tutorial. Stay tuned by signing up for [my newsletter](http://eepurl.com/gxmy39). If you have any questions/comments/proposals, feel free to shoot me a message on [Twitter](https://twitter.com/0xboz)/[Discord](https://discord.gg/jchMcc2)/[Patreon](https://www.patreon.com/0xboz). Happy coding!