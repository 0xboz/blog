---
title: "How to Use VirtualBox As a Remote PostgreSQL Server"
date: 2019-08-01
tags: [VirtualBox, PostgreSQL, Debian]
excerpt: "Use VirtualBox as a remote PostgreSQL server"
header:
    image: /assets/images/cold_high_angle_shot_motion.jpg
    caption: "Photo credit: [**Muffin**](https://www.pexels.com/@muffin)"
---

I think this would be an interesting post about creating a "remote" [PostgreSQL](https://www.postgresql.org/) server by [VirtualBox](https://www.virtualbox.org/). Both guest and host machines are Debian. 

### VB Guest Debian Installation (```netinst```)
[Download Debian netinst cd](https://www.debian.org/distrib/netinst#netboot). 

The installation is pretty straightforward. There is only one personal preference I would like to make - Deselect all extra packages. After all, this is just a remote server, we don't need GUI or anything extra fancy.

<figure>
    <a href="{{ site.url }}{{ site.baseurl }}/assets/images/unselect_extra_debian_packages.png">
        <img src="{{ site.url }}{{ site.baseurl }}/assets/images/unselect_extra_debian_packages.png">
    </a>
    <figcaption>Deselect all extra software</figcaption>
</figure>

### Network
One important step in VB setup is to switch the default network from NAT to Bridged Adapter. If you would like to know the difference, here is the official doc on [this topic](https://www.virtualbox.org/manual/ch06.html).

### ```sudo``` Group
Log into VB guest Debian with your credentials, add yourself to sudo group and reboot.

```
su
apt install sudo
/usr/sbin/adduser bo sudo 
/usr/sbin/reboot
```

### PostgreSQL Installation in VB Guest Debian
When you have logged back in, run the command below to install PostgreSQL

```
sudo apt update
sudo apt install -y postgresql postgresql-contrib
```

Check if everything works ok.
```
sudo service postgresql status
```

By default, the installation process will create a superuser named ```postgres```. This particular account is recommended to be used locally without password.

To access this account locally, you have two options:
* 1. Switch to postgres account and type psql

```
sudo su - postgres
psql
```

* 2. Run this command 

```
sudo -u postgres psql
```

To exit out of PostgreSQL shell, type ```\q``` or ```Ctrl + D```.

### Create Roles and Database

Within the VB guest Debian, type

```
sudo su - postgres -c "createuser bo"
sudo su - postgres -c "createdb bodb"
```

Connect PostgreSQL shell and grant privileges

```
sudo -u postgres psql
```

In PostgreSQL shell, type

```
grant all privileges on database bodb to bo;
```

### Enable Remote Access
In the terminal, run the following command

```
sudo nano /etc/postgresql/11/main/postgresql.conf
```

Find the line below by ```Ctrl + w```, and change it accordingly.

*listen_addresses = '\*'     # what IP address(es) to listen on;*

Restart PostgreSQL

```
sudo service postgresql restart
```

Accept remote connections 

```
sudo nano /etc/postgresql/11/main/pg_hba.conf
```

Add this line to pg_hba.conf  

*host    bodb             bo            0.0.0.0/0            trust*

One more thing before we move on to the host machine, we need to find out the IP address of the guest machine on our local network.

```
ip address
```

Mine is 192.168.1.24

### Host Machine Debian
We need to install the client package from the repo.

```
sudo apt install postgresql-client
```

Now we can connect to our "remote" postsql server by running this command

```
psql -U bo -d bodb -h 192.168.1.24
```

Voilà! 

<figure>
    <a href="{{ site.url }}{{ site.baseurl }}/assets/images/connect_remote_vb_postgresql.png">
        <img src="{{ site.url }}{{ site.baseurl }}/assets/images/connect_remote_vb_postgresql.png">
    </a>
    <figcaption>Connect to VirtualBox PostgreSQL server. (Since the host machine is Debian 9 and guest machine is Debian 10, we have different versions of PostgreSQL installed. Other than that, everything should be fine.)</figcaption>
</figure>

I hope you liked this short tutorial. Stay tuned by signing up for [my newsletter](http://eepurl.com/gxmy39). If you have any questions/comments/proposals, feel free to shoot me a message on [Twitter](https://twitter.com/0xboz)/[Discord](https://discord.gg/jchMcc2)/[Patreon](https://www.patreon.com/0xboz). Happy coding!