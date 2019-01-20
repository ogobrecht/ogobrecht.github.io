---
title: Pitfalls with Oracle 11g XE and Docker on Mac OS
description: The swap space problem and how to fix it
tags: [docker, oracle, mac]
lang: en
publishdate: 2017-03-21
lastmod: 2017-08-12 20:12:00
---

Recently I tried to install Oracle XE into a docker container on Mac OS Sierra using the [official docker file][1] from Oracle. I had no luck, because the installation failed with this error message:

> This system does not meet the minimum requirements for swap space. Based on the amount of physical memory available on the system, Oracle Database 11g Express Edition requires 2048 MB of swap space. This system has 0 MB of swap space. Configure more swap space on the system and retry the installation.

The problem is, that you can't do anything for this on a Mac - the system manages the swap files by itself and if you need for an example 64 GB you will get it. My new standard MacBook has 8 GB RAM and there is currently no reason for a swap file.

So, what can we do? After a short search I found [this blog post][2], which describes how to alter the pre-installation script of the RPM installation file with the help of the `repmrebuild` command. Unfortunately this command is not available on Mac OS. I found only rpmbuild as part of the rpm installation with [Homebrew][3].

Luckily I found another way to modify a rpm package under Mac OS: There is a Ruby based tool to build packages for multiple platforms called [fpm][4]. So, lets start:

If you don't have [Homebrew][3] on your Mac, install this first:

```bash
/usr/bin/ruby -e \
"$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Install rpm:

```bash
brew install rpm
```

Install tar:

```bash
brew install gnu-tar
```

Install fpm:

```bash
gem install --no-ri --no-rdoc fpm
```

Unzip the downloaded Linux XE installation archive and go into the directory Disk1:

```bash
unzip oracle-xe-11.2.0-1.0.x86_64.rpm.zip && \
cd Disk1
```

Write rpm scripts to text file:

```bash
rpm \
  --scripts \
  -qp oracle-xe-11.2.0-1.0.x86_64.rpm \
  > scripts.txt
```

Find this loop (from line 197 until line 212 as of this writing) and comment out or delete it:

```bash
# check and disallow install, if swap space is less than Min( 2047, 2 * RAM)`
if [ $swapspace -lt $requiredswapspace ]
then
	if [ "$requiredswapspace" = "2047" ];
	then
		requiredswapspace=2048
	fi
	echo
        echo "This system does not meet the minimum requirements for swap space.  Based on
the amount of physical memory available on the system, Oracle Database 11g
Express Edition requires $requiredswapspace MB of swap space. This system has $swapspace MB
of swap space.  Configure more swap space on the system and retry the
installation."
	echo
        exit 1
fi
```

Find this line (starting on line 328 as of this writing) and delete the line and everything behind until the file end:

```bash
postinstall scriptlet (using /bin/sh):
```

Delete the very first line containing this code:

```bash
preinstall scriptlet (using /bin/sh):
```

Save the file and bring it back into the rpm with this command (will take some time):

```bash
fpm \
  -s rpm \
  -t rpm \
  -f --before-install scripts.txt \
  ./oracle-xe-11.2.0-1.0.x86_64.rpm
```

Go out of Disk1 and update the zip archive, because the Oracle provided build script needs it compressed (will take some time):

```bash
cd .. && \
zip -r oracle-xe-11.2.0-1.0.x86_64.rpm.zip Disk1
```

When you now do the Docker build you will face another error message:

> package oracle-xe-11.2.0-1.0.x86_64 is intended for a different operating system

To fix this is fairly easy. Open the Dockerfile.xe, go to line 61 (as of this writing) and change this code:

```bash
rpm -i Disk1/*.rpm &&    \
```
to this one:

```bash
rpm -i --ignoreos Disk1/*.rpm && \
```

When you now start the docker build you should be successful (will take some time):

```bash
sudo ./buildDockerImage.sh -v 11.2.0.2 -x -i
```

The last step is to run the container - on the first startup the database is created (will take some time, align the name to your needs):

```bash
docker run \
  --name orclapex51 \
  --shm-size=1g \
  -p 1521:1521 \
  -p 8080:8080 \
  oracle/database:11.2.0.2-xe
```

If you plan to use APEX without any dedicated webserver (maybe because you are the only user and it is only a small dev instance) you should pimp the EPG a little bit:

```sql
alter system set shared_servers=10 scope=both;
alter system set max_shared_servers=20 scope=both;
alter system set dispatchers='(PROTOCOL=TCP) (SERVICE=xeXDB) (DISPATCHERS=2)' scope=both;
```

I use currently a Node.js based proxy server running directly on my Mac OS, because I need a webserver there anyway. The nice thing is, that the EPG don't need to serve the images and I also don't need to install the images into the EPG, when I upgrade to a newer APEX version. This saves me time and the EPG runs faster because of less number of requests. I will cover this topic in the next post.

Happy installing :-)  
Ottmar

[1]: https://github.com/oracle/docker-images/blob/master/OracleDatabase/dockerfiles/11.2.0.2/Dockerfile.xe
[2]: https://www.elastichosts.com/blog/oracle-database-installation-on-a-container-running-centos/
[3]: https://brew.sh
[4]: https://github.com/jordansissel/fpm
