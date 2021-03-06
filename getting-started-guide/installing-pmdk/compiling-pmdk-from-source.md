# Installing PMDK from Source on Linux

## Overview

This procedure describes how to clone the source code from the pmdk github repository and compile, then install it.

{% hint style="info" %}
**Note:** We recommend [installing NDCTL](../installing-ndctl.md) first so PMDK builds all features. If the ndctl development packages and header files are not installed, PMDK will build successfully, but will disable some of the RAS \(Reliability, Availability and Serviceability\) features.
{% endhint %}

If your system is behind a firewall and requires a proxy to access the Internet, configure your package manager to use a proxy.

## Install Prerequisites

To build the PMDK libraries on Linux, you may need to install the following required packages on the build system:

* autoconf
* automake
* gcc
* gcc-c++
* glib2-devel
* libfabric-devel
* pandoc
* pkg-config
* ncurses-devel

{% tabs %}
{% tab title="Fedora" %}
```text
$ sudo dnf install autoconf automake pkg-config glib2-devel libfabric-devel pandoc ncurses-devel
```
{% endtab %}

{% tab title="RHEL/CentOS" %}
Some of the required packages can be found in the EPEL repository. Verify the EPEL repository is active:

```text
$ sudo yum repolist
```

If the EPEL repository is not listed, install and activate it using:

```text
$ sudo yum -y install epel-release
```

To install the prerequisite packages, run:

```text
$ sudo yum install autoconf automake pkgconfig glib2-devel libfabric-devel pandoc ncurses-devel
```
{% endtab %}

{% tab title="Ubuntu & Debian" %}
**For Ubuntu 18.04 \(Bionic\) or Debian 9 \(Stretch\) or later**

```text
$ sudo apt install autoconf automake pkg-config libglib2.0-dev libfabric-dev pandoc libncurses5-dev
```

**For Ubuntu 16.04 \(Xenial\) and Debian 8 \(Jessie\):**

{% hint style="info" %}
Earlier releases of Ubuntu and Debian do not have libfabric-dev available in the repository. If this library is required, you should compile it yourself. See [https://github.com/ofiwg/libfabric](https://github.com/ofiwg/libfabric)
{% endhint %}

```text
$ sudo apt install autoconf automake pkg-config libglib2.0-dev pandoc libncurses5-dev
```

\*\*\*\*
{% endtab %}

{% tab title="FreeBSD" %}
To build and test the PMDK library on FreeBSD, you may need to install the following required packages on the build system:

* autoconf
* bash
* binutils
* coreutils
* e2fsprogs-libuuid
* gmake
* glib2
* glib2-devel
* libunwind
* ncurses\*
* pandoc
* pkgconf

```text
$ sudo pkg install autoconf automake bash coreutils e2fsprogs-libuuid glib2 glib2-devel gmake libunwind pandoc ncurses pkg-config
```

\(\*\) The pkg version of ncurses is required for proper operation; the base version included in FreeBSD is not sufficient.
{% endtab %}
{% endtabs %}

The `git` utility is required to clone the repository or you can download the source code as a [zip file](https://github.com/pmem/pmdk/archive/master.zip) directly from the [repository ](https://github.com/pmem/pmdk)on GitHub.

### Optional Prerequisites

The following packages are required only by selected PMDK components or features. If not present, those components or features may not be available:

* **libfabric** \(v1.4.2 or later\) -- required by **librpmem**
* **libndctl** and **libdaxctl** \(v60.1 or later\) -- required by **daxio** and RAS features.  See [Installing NDCTL](../installing-ndctl.md)
  * To build pmdk without ndctl support, set 'NDCTL\_ENABLE=n' using: `$ export NDCTL_ENABLE=n`

### Compiler Requirements

A C/C++ Compiler is required. GCC/G++ will be used in this documentation but you may use a different compiler then set the `CC` and `CXX` shell environments accordingly.

{% tabs %}
{% tab title="Fedora" %}
```text
$ sudo dnf install gcc gcc-c++
```
{% endtab %}

{% tab title="RHELCentOS" %}
```text
$ sudo yum install gcc gcc-c++
```
{% endtab %}

{% tab title="Ubuntu/Debian" %}
```text
$ sudo apt install gcc g++
```
{% endtab %}
{% endtabs %}

## Clone the PMDK GitHub Repository

The following uses the `git` utility to clone the repository.

```text
$ git clone https://github.com/pmem/pmdk
$ cd pmdk
```

Alternatively you may download the source code as a [zip file](https://github.com/pmem/pmdk/archive/master.zip) from the [GitHub website](https://github.com/pmem/pmdk).

```text
$ wget https://github.com/pmem/pmdk/archive/master.zip
$ unzip master.zip
$ cd pmdk-master
```

## Compile

{% tabs %}
{% tab title="Linux & FreeBSD" %}
To build the master branch run the `make` utility in the root directory:

```text
$ make
```

If you want to compile with a different compiler, you have to provide the `CC` and `CXX`variables. For example:

```text
$ make CC=clang CXX=clang++
```

These variables are independent and setting `CC=clang` does not set `CXX=clang++`.

{% hint style="info" %}
If the `make` command returns an error similar to the following, this is caused by pkg-config being unable to find the required "libndctl.pc" file.

```text
$ make
src/common.inc:370: *** libndctl(version >= 60.1) is missing -- see README.  Stop.
```

This can occur when libndctl was installed in a directory other than the /usr location.

To resolve this issue, the PKG\_CONFIG\_PATH is a environment variable that specifies additional paths in which pkg-config will search for its .pc files.

This variable is used to augment pkg-config's default search path. On a typical Unix system, it will search in the directories /usr/lib/pkgconfig and /usr/share/pkgconfig. This will usually cover system installed modules. However, some local modules may be installed in a different prefix such as /usr/local. In that case, it's necessary to prepend the search path so that pkg-config can locate the .pc files.

The pkg-config program is used to retrieve information about installed libraries in the system. The primary use of pkg-config is to provide the necessary details for compiling and linking a program to a library. This metadata is stored in pkg-config files. These files have the suffix .pc and reside in specific locations known to the pkg-config tool.

To check the PKG\_CONFIG\_PATH value use this command:

`$ echo $PKG_CONFIG_PATH`

To set the PKG\_CONFIG\_PATH value use:

```text
$ export PKG_CONFIG_PATH=/usr/local/lib64/pkgconfig:/usr/local/lib/pkgconfig:${PKG_CONFIG_PATH}
```

Now execute the `make` command again.
{% endhint %}
{% endtab %}
{% endtabs %}

## Install

Installing the library is more convenient since it installs man pages and libraries in the standard system locations.

{% tabs %}
{% tab title="Linux & FreeBSD" %}
To install the libraries to the default `/usr/local` location:

```text
$ sudo make install
```

To install this library into other locations, you can use the `prefix=path` option, e.g:

```text
$ sudo make install prefix=/usr
```

If you installed to non-standard directory \(anything other than /usr\) you may need to add $prefix/lib or $prefix/lib64 \(depending on the distribution you use\) to the list of directories searched by the linker:

```text
sudo sh -c "echo /usr/local/lib >> /etc/ld.so.conf"
sudo sh -c "echo /usr/local/lib64 >> /etc/ld.so.conf"
sudo ldconfig
```
{% endtab %}
{% endtabs %}

