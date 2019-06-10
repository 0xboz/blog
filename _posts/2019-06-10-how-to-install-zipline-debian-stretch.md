---
title: "How to Install Zipline on Debian 9 Stretch"
date: 2019-06-10
tags: [Zipline, Debian, Conda]
excerpt: "Zipline installation on Debian 9 (Stretch)"
---

Apparently, we need to install Zipline first before we can test our trading ideas. There are many tutorials to teach you how to install Zipline in a variety of ways. In this post, I am going to show you my installation on Debian 9 Stretch.

And of course, here is [the official document](http://www.zipline.io/install.html) from Zipline project.

First, let us get dependencies out of the way.
```
sudo apt install -y libatlas-base-dev python-dev gfortran pkg-config libfreetype6-dev
```

Since more complex issues can occur via pip installation route, I have taken on *the conda way*. More details about anaconda installation on Debian also can be found [here](https://docs.anaconda.com/anaconda/install/linux/).

Extended dependencies for GUI packages
```
sudo apt-get -y install libgl1-mesa-glx libegl1-mesa libxrandr2 libxrandr2 libxss1 libxcursor1 libxcomposite1 libasound2 libxi6 libxtst6
```
Ok, let us get down to it.
```
cd ~/Downloads
wget https://repo.anaconda.com/archive/Anaconda3-2019.03-Linux-x86_64.sh
bash ~/Downloads/Anaconda3-2019.03-Linux-x86_64.sh
```
This step might take a few minutes. It depends on your internet speed. After it is done, close the terminal and start a new one. If you would like to deactivate the base environment by default, use this command.
```
conda config --set auto_activate_base False
```
How about launching anaconda-navigator from the application menu? You can use a menu editor like alacarte to add a new item.
```
sudo apt install -y alacarte
```
The default location of the app should be ```$HOME/anaconda3/bin/anaconda-navigator```

Great! We now create a virtual env named ```zipline``` using conda.
```
conda create -n zipline python=3.5
```
Activate this env and install Zipline via Quantopian channel
```
conda install -c Quantopian zipline
```
I would also like to have jupyterlab and jupyter notebook in this env
```
conda install -c conda-forge jupyterlab
```
*Note: If you install jupyterlab via the default channel, conda might switch Python to 2.7 for some reasons.*

We are almost done, but there are a few known issues I have encountered along the way. I think this is the best place to address those all before we run into them later on.

> ImportError: No Module named 'talib'

In [one](http://www.zipline.io/beginner-tutorial.html#my-first-algorithm) of the official examples, we might see a problem like this.
```
from zipline.examples import buyapple
```
![zipline talib issue](/images/zipline_talib_issue.png)
To mitigate this issue, we can install ta-lib via Quantopian channel
```
conda install -c Quantopian ta-lib
```

> Ignoring ? values because they are out of bounds for uint32

I got to know this one when importing large trading volumes from crypto data. Here are some discussions about [this topic](https://github.com/quantopian/zipline/issues/1572). After Trial-and-error, I have temporarily disabled this warning and save the missing data.

First, go to the zipline package folder. If you have followed this tutorial along, it should be located at ```$HOME/anaconda3/envs/zipline/lib/python3.5/site-packages/zipline/data```

Fire up your favorite text editor and replace 'uint32' with 'uint64' in those two files: ```us_equity_pricing.py``` and ```minute_bars.py```

I will try to update more known Zipline issues and their solutions in the near future. Stay tuned and Ciao!