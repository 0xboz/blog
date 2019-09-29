---
title: "How to Run ScyllaDB on VirtualBox Guest Debian"
date: 2019-09-29
tags: [ScyllaDB, VirtualBox, Debian]
excerpt: "Test ScyllaDB on VirtualBox Guest Debian"
header:
    image: /assets/images/blur_color.jpg
    caption: "Photo credit: [**Karol D**](https://www.pexels.com/@karoldach)"
---

I came across [ScyllaDB](https://www.scylladb.com/) while I was researching how to store tick data for [backtesting](https://www.investopedia.com/terms/b/backtesting.asp).


<div class="notice--info">
  <p>Tick data refers to any market data which shows the price and volume of every print.  Additionally tick data often includes information about every change to the best bid and ask.  The most common alternative to tick data is candlestick data.  Although the charts may be drawn different ways, it is common to compress tick data by grouping the ticks into candlesticks.  This compressed data is more commonly available than tick data because it requires significantly less memory, bandwidth, disk space, etc.  In addition to the computer hardware involved, most people can only take in a limited amount of tick data at one time.</p>
</div>

Conventionally, traders might use relational database like [MySQL](https://www.mysql.com/) and [PostgreSQL](https://www.postgresql.org/) to get the job done. I believe there might be a better alternative to deal with Time Series data, for instance, [OpenTSDB](http://opentsdb.net/) and [Apache Cassandra](https://cassandra.apache.org/).

I have [a similar post](https://0xboz.github.io/blog/how-to-use-virtualbox-as-a-remote-postgresql-server/) about using VirtualBox as a remote PostgreSQL server. In this post, however, we are going to explore the setup of the C++ cousin of Apache Cassandra - ScyllaDB on VirtualBox guest Debian. [The official documentation](https://www.scylladb.com/download/open-source/scylla-virtualbox/) of ScyllaDB has a test-drive on VirtualBox. However, they have only mentioned the setup for [RHEL](https://www.redhat.com/) and [CentOS](https://centos.org/) for some reason. I hope this tutorial will serve as a complimentary doc for that purpose. 

### VB Guest Debian Installation (```netinst```)
[Download Debian netinst cd](https://www.debian.org/distrib/netinst#netboot). 

The installation is pretty straightforward. There is only one personal preference I would like to make - Deselect all extra packages. After all, this is just a remote server, we don't need GUI or anything extra fancy.

<figure>
    <a href="{{ site.url }}{{ site.baseurl }}/assets/images/unselect_extra_debian_packages.png">
        <img src="{{ site.url }}{{ site.baseurl }}/assets/images/unselect_extra_debian_packages.png">
    </a>
    <figcaption>Deselect all extra software</figcaption>
</figure>

After installation, make sure you have `sudo` ready.

```
su
apt install -y sudo
adduser YOUR_USER_NAME sudo
reboot
```

### Network
Comparing with the tutorial for PostgreSQL on VirtualBox, we will use NAT instead of Bridge network this time.  

Define forwarding ports in the NAT configuration in VirtualBox network settings: Check your network settings and define port forwarding as described below: Machine → Settings → Network → Port Forwarding

<figure>
    <a href="{{ site.url }}{{ site.baseurl }}/assets/images/check_virtualbox_network_settings.png">
        <img src="{{ site.url }}{{ site.baseurl }}/assets/images/check_virtualbox_network_settings.png">
    </a>
    <figcaption>Check VirtualBox network settings</figcaption>
</figure>

Set the forwarding ports for Scylla: 9042, 7000, 7001, 9180 and 7199

<figure>
    <a href="{{ site.url }}{{ site.baseurl }}/assets/images/set_forwarding_rules.png">
        <img src="{{ site.url }}{{ site.baseurl }}/assets/images/set_forwarding_rules.png">
    </a>
    <figcaption>Set forwarding rules</figcaption>
</figure>

### Install ScyllaDB

Install dependencies
```
sudo apt-get install apt-transport-https wget gnupg2 dirmngr
sudo apt-get update
```

Set Keys
```
sudo apt-key adv --fetch-keys https://download.opensuse.org/repositories/home:/scylladb:/scylla-3rdparty-stretch/Debian_9.0/Release.key

sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 17723034C56D4B19
```

Add the Scylla APT repository
```
sudo wget -O /etc/apt/sources.list.d/scylla.list http://repositories.scylladb.com/scylla/repo/546ab40d-5c6e-4085-ac94-deafdce9ca87/debian/scylladb-3.0-stretch.list
```

Install ScyllaDB
```
sudo apt-get update
sudo apt-get install scylla
```

### Configuration

Configure the `/etc/scylla/scylla.yaml` file with the following parameters:

```
...
cluster_name: 'VirtualBox ScyllaDB Cluster'
...
listen_address: 10.0.2.15
...
rpc_address: 10.0.2.15
...
- seeds: "10.0.2.15"
...
```

### Start ScyllaDB

```
sudo scylla_io_setup
sudo systemctl start scylla-server.service
```

Connect to ScyllaDB via cqlsh
```
cqlsh 10.0.2.15
```  

I hope you have enjoyed this short tutorial. 

Stay tuned by signing up for [my newsletter](http://eepurl.com/gxmy39). If you have any questions/comments/proposals, feel free to shoot me a message on [Twitter](https://twitter.com/0xboz)/[Discord](https://discord.gg/JHt7UQu)/[Patreon](https://www.patreon.com/0xboz). Happy Trading!
