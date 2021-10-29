---
author: "Brandon Erik Bertelsen"
title: "Installing a newer version of gdal on Ubuntu 20.04"
date: "2015-06-15"
categories:
- R
---

The installation candidate for GDAL for Ubuntu 20.04 is quite old. The mapviews team has come up with some rather good new features that rely on v3.3.1 or newer. 

First, use the PPA from ubuntugis-unstable:

```{shell}
sudo add-apt-repository ppa:ubuntugis/ubuntugis-unstable
# enter

sudo apt-get update
```

You must purge existing versions of gdal-bin, libkml, and libgdal on your machine. Note that your version numbers may vary (libgdal29, for example, may be something else)

```{shell}
sudo apt-get purge --auto-remove libgdal
sudo apt-get purge --auto-remove libgdal29
sudo apt-get purge --auto-remove gdal-bin
sudo apt-get purge --auto-remove libkml-dev
sudo apt-get purge --auto-remove libproj-dev

sudo find /usr/local/bin/ -name 'gdal*' -delete
sudo find /usr/local/lib/ -name 'libgdal*' -delete
```

Install the version of the package from unstable ppa. 

```{shell}
sudo apt-get install libgdal-dev gdal-bin libproj15 libproj19 libproj-dev
```

Now there are some problems with the libs from the unstable package but they are fairly straightforward to figure out. THe main issue is that some of the libs point to older versions of themselves or point to themselves in the wrong place. 

When attempting to install `sf` in R using `install.packages('sf')` you may encounter the following error: 

```
unable to load shared object '/usr/local/lib/R/site-library/terra/libs/terra.so': libproj.so.12: cannot open shared object file: No such file or directory
```

This is telling you that for whatever reason it can not find libproj.so.12. You can correct this by creating a symbolic link between the version of libproj you are actually using and this older one. In my case, this was libproj15

```
sudo ln -s /usr/lib/x86_64-linux-gnu/libproj.so.15 /usr/lib/libproj.so.12
```

You may experience any number of these "can't find the lib durrrr" kind of problems. In my experience they were solved by first finding the actual lib available on my system and then symlinking them to the version desired by the R packages installation process.

In my case, the R package installation was looking in `/usr/lib/` but it was actually in `/usr/lib/x86_64-linux-gnu/`. You don't need to remember this, you can check for yourself with the following:

```
$ dpkg -L libproj15
/.
/usr
/usr/lib
/usr/lib/x86_64-linux-gnu
/usr/lib/x86_64-linux-gnu/libproj.so.15.3.1
/usr/share
/usr/share/doc
/usr/share/doc/libproj15
/usr/share/doc/libproj15/AUTHORS
/usr/share/doc/libproj15/NEWS.Debian.gz
/usr/share/doc/libproj15/NEWS.gz
/usr/share/doc/libproj15/README.gz
/usr/share/doc/libproj15/changelog.Debian.gz
/usr/share/doc/libproj15/copyright
/usr/lib/x86_64-linux-gnu/libproj.so.15
```

Looking at the end there we see the lib. Just remember that creating the symlink is in the form `sudo ln -s source target`. Where source is what you would find in `dpkg -L your_lib` and target is what the R installation process complains about missing.
