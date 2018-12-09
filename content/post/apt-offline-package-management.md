---
title: "Aptitude Offline Package Management"
date: 2018-12-09T20:08:20+03:00
lastmod: 2018-12-09T20:08:20+03:00
draft: false
keywords: ["apt-offline", "apt", "aptitude", "offline package management", "apt offline", "ubuntu offline package management", "debian offline package management"]
description: "Offline package management for APT"
tags: ["linux", "apt-offline", "apt", "tips"]
categories: ["devops", "system","tips", "linux"]
author: "Aziz Unsal"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---
Sometimes you need to use a package when there is no internet access. But connecting to the Internet and using the lovely `aptitude` are not possible for the server(s) you want to make changes due to network restrictions!

`apt-offline` to the rescue. Come with me.
<!--more-->

# A Real World Scenario
---
In my case, `cifs-utils` was the missing package. Although I configured the servers with the required packages, I had forgotten that the production environment was using CIFS instead of NFS. 

I changed the `/etc/fstab` pre-defined entries with the correct ones for the CIFS share then ran the command below to refresh the mount points;
```
$ sudo mount -a
```
![And boom!](/blog/boom.png)


> mount: wrong fs type, bad option, bad superblock on XXXX, missing codepage or helper program, or other error (for several filesystems (e.g. nfs, cifs) you might need a /sbin/mount.<type> helper program)  
In some cases useful info is found in syslog - try dmesg | tail or so.


The error message gives us the clue;

>missing codepage or helper program, or other error (for several filesystems (e.g. nfs, cifs) you might need a /sbin/mount.<type> helper program)  

As I said the servers were configured to create NFS mounts. Therefore the *helper program* in which mentioned in the error message should be the other one, the `cifs`.

Normally you can easily solve that problem issuing an aptitude install command;

```
$ sudo apt install cifs-utils
``` 

Here we are, let's talk about the purpose of this blog post; installing a package without internet access. 

# A Quick apt-offline Introduction
---

Using [apt-offline](https://packages.ubuntu.com/xenial/admin/apt-offline) is a bit complicated, at least it was difficult for me when the first time I used it. However, we can go step by step with an example. This example will be the missing package I mentioned in the previous paragraph; `cifs-utils`.

## Create a Signature
```
$ sudo apt-offline set cifs-utils.sig --install-packages cifs-utils
```

## Download the Package Based On the Created Signature

```
$ ls -l
total 4
-rwxrwxr-x 1 ubuntu-template ubuntu-template 3642 Dec  7 20:04 cifs-utils.sig

$ apt-offline get cifs-utils.sig --bundle ./cifs-utils.zip

Fetching APT Data

Downloading libwbclient0 2%3a4.3.11+dfsg-0ubuntu0.16.04.18 - 29 KiB
libwbclient0 2%3a4.3.11+dfsg-0ubuntu0.16.04.18 done.
Downloading samba-common 2%3a4.3.11+dfsg-0ubuntu0.16.04.18 - 82 KiB
samba-common 2%3a4.3.11+dfsg-0ubuntu0.16.04.18 done.
Downloading libtalloc2 2.1.5-2 - 33 KiB
libtalloc2 2.1.5-2 done.
Downloading cifs-utils 2%3a6.4-1ubuntu1.1 - 72 KiB
cifs-utils 2%3a6.4-1ubuntu1.1 done.
Downloading libavahi-common-data 0.6.32~rc+dfsg-1ubuntu2.2 - 21 KiB
libavahi-common-data 0.6.32~rc+dfsg-1ubuntu2.2 done.
Downloading libavahi-common3 0.6.32~rc+dfsg-1ubuntu2.2 - 21 KiB
libavahi-common3 0.6.32~rc+dfsg-1ubuntu2.2 done.
Downloading libavahi-client3 0.6.32~rc+dfsg-1ubuntu2.2 - 24 KiB
libavahi-client3 0.6.32~rc+dfsg-1ubuntu2.2 done.
Downloading libcups2 2.1.3-4ubuntu0.5 - 192 KiB
libcups2 2.1.3-4ubuntu0.5 done.
Downloading libtdb1 1.3.8-2 - 37 KiB
libtdb1 1.3.8-2 done.
Downloading libtevent0 0.9.28-0ubuntu0.16.04.1 - 25 KiB
libtevent0 0.9.28-0ubuntu0.16.04.1 done.
Downloading libldb1 2%3a1.1.24-1ubuntu3 - 105 KiB
libldb1 2%3a1.1.24-1ubuntu3 done.
Downloading libpython2.7 2.7.12-1ubuntu0~16.04.4 - 1 MiB
libpython2.7 2.7.12-1ubuntu0~16.04.4 done.
Downloading python-crypto 2.6.1-6ubuntu0.16.04.3 - 240 KiB
python-crypto 2.6.1-6ubuntu0.16.04.3 done.
Downloading python-ldb 2%3a1.1.24-1ubuntu3 - 28 KiB
python-ldb 2%3a1.1.24-1ubuntu3 done.
Downloading python-tdb 1.3.8-2 - 10 KiB
python-tdb 1.3.8-2 done.
Downloading python-talloc 2.1.5-2 - 7 KiB
python-talloc 2.1.5-2 done.
Downloading samba-libs 2%3a4.3.11+dfsg-0ubuntu0.16.04.18 - 4 MiB
samba-libs 2%3a4.3.11+dfsg-0ubuntu0.16.04.18 done.
Downloading python-samba 2%3a4.3.11+dfsg-0ubuntu0.16.04.18 - 1 MiB
python-samba 2%3a4.3.11+dfsg-0ubuntu0.16.04.18 done.
Downloading samba-common-bin 2%3a4.3.11+dfsg-0ubuntu0.16.04.18 - 493 KiB
samba-common-bin 2%3a4.3.11+dfsg-0ubuntu0.16.04.18 done.
 19 /  19 items: [##############################] 100.0% of 8 MiB
Downloaded data to /home/ubuntu-template/cifs-utils.zip
```

As you see, `apt-offline` downloaded with all of its required dependencies not only `cifs-utils`. Wonderful!

## Update APT Database With Copied Data
```
$ ls -l
total 8564
-rwxrwxr-x 1 ubuntu-template ubuntu-template    3642 Dec  7 20:04 cifs-utils.sig
-rw-rw-r-- 1 ubuntu-template ubuntu-template 8764161 Dec  9 22:25 cifs-utils.zip

$ sudo apt-offline install cifs-utils.zip
libwbclient0_2%3a4.3.11+dfsg-0ubuntu0.16.04.18_amd64.deb file synced.
samba-common_2%3a4.3.11+dfsg-0ubuntu0.16.04.18_all.deb file synced.
libtalloc2_2.1.5-2_amd64.deb file synced.
cifs-utils_2%3a6.4-1ubuntu1.1_amd64.deb file synced.
libavahi-common-data_0.6.32~rc+dfsg-1ubuntu2.2_amd64.deb file synced.
libavahi-common3_0.6.32~rc+dfsg-1ubuntu2.2_amd64.deb file synced.
libavahi-client3_0.6.32~rc+dfsg-1ubuntu2.2_amd64.deb file synced.
libcups2_2.1.3-4ubuntu0.5_amd64.deb file synced.
libtdb1_1.3.8-2_amd64.deb file synced.
libtevent0_0.9.28-0ubuntu0.16.04.1_amd64.deb file synced.
libldb1_2%3a1.1.24-1ubuntu3_amd64.deb file synced.
libpython2.7_2.7.12-1ubuntu0~16.04.4_amd64.deb file synced.
python-crypto_2.6.1-6ubuntu0.16.04.3_amd64.deb file synced.
python-ldb_2%3a1.1.24-1ubuntu3_amd64.deb file synced.
python-tdb_1.3.8-2_amd64.deb file synced.
python-talloc_2.1.5-2_amd64.deb file synced.
samba-libs_2%3a4.3.11+dfsg-0ubuntu0.16.04.18_amd64.deb file synced.
python-samba_2%3a4.3.11+dfsg-0ubuntu0.16.04.18_amd64.deb file synced.
samba-common-bin_2%3a4.3.11+dfsg-0ubuntu0.16.04.18_amd64.deb file synced.
```

## Install the Package
```
$ sudo apt install cifs-utils
```

And let's check whether the package is installed or not;

```
$ sudo dpkg -l cifs-utils
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                                                  Version                         Architecture                    Description
+++-=====================================================-===============================-===============================-================================================================================================================
ii  cifs-utils                                            2:6.4-1ubuntu1.1                amd64                           Common Internet File System utilities
```


We are done, the missing package has been installed successfully!

## Recap
---
1. Create a signature *(offline machine)*; `$ sudo apt-offline get cifs-utils.sig --bundle ./cifs-utils.zip`
2. Download the package based on the created signature *(online machine)*; `$ sudo apt-offline get cifs-utils.sig --bundle ./cifs-utils.zip`
3. Update APT database with copied data *(offline machine)*; `$ sudo apt-offline install cifs-utils.zip`
4. Install the package *(offline machine)*; `$ sudo apt install cifs-utils`


I highly recommend you to include `apt-offline` package in your toolbox, especially if you are going to work in a restricted environment. It was one of my colleagues who let me know this handy package.