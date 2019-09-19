---
title: "How to Install Catalyst on Debian 9 Stretch"
date: 2019-06-18
tags: [Catalyst, Debian]
excerpt: "Catalyst installation on Debian 9 (Stretch)"
header:
    image: /assets/images/mountain_dawn.jpg
---
[Catalyst](https://enigma.co/catalyst/index.html), a fork of [Zipline](http://www.zipline.io/), is an algorithmic trading library for crypto-assets.

In this tutorial, I am going to show you how to get it running on Debian 9 Stretch via pip. First, let us create a virtual environment.
```
mkdir catalyst
cd catalyst
python3 -m venv venv
```
Next, we need to install some essential dependencies in our OS.
```
sudo apt install -y libatlas-base-dev python-dev python3-dev gfortran pkg-config libfreetype6-dev && sudo apt install --reinstall build-essential
```
Great! Activate our venv and you will be on your way shortly.
```
source venv/bin/activate
(venv) pip install -U pip setuptools
(venv) pip install enigma-catalyst matplotlib
```
When it is done, we can also give a quick check if everything works.
```
(venv) python
Python 3.5.3 (default, Sep 27 2018, 17:25:39) 
[GCC 6.3.0 20170516] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from catalyst import run_algorithm
>>> 
```
Awesome! I hope this short post can be helpful to someone who is new to Catalyst. Feel like stopping by and say hi? Let us talk more about crypto and quantitative trading over there. [Here is the discord invite link](https://discord.gg/JHt7UQu). 