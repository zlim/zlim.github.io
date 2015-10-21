---
layout:	post
title:	"Optimizing Theano on Mac"
date:	2015-10-21 00:17:02
tags:	[performance, mac, theano]
---
In this post, we look into speeding up [Theano](http://deeplearning.net/software/theano/) on Mac, by enabling use of multiple CPU threads. The basic idea is to enable OpenMP so that Theano can do parallel processing on multiple cores.

For instructions of how to install Theano on Mac, please refer to the official [documentation](http://deeplearning.net/software/theano/install.html#homebrew).

When attempting to tune Theano per ["Multi cores support in Theano"](http://deeplearning.net/software/theano/tutorial/multi_cores.html), we quickly run into an error on Mac.
{% highlight bash %}
$ python theano/misc/elemwise_openmp_speedup.py
/Users/z/Library/Python/2.7/lib/python/site-packages/theano/tensor/opt.py:4684: UserWarning: Your g++ compiler fails to compile OpenMP code. We know this happen with some version of the EPD mingw compiler and LLVM compiler on Mac OS X. We disable openmp everywhere in Theano. To remove this warning set the theano flags `openmp` to False.
  no_recycling=[])

{% endhighlight %}
This is because Xcode ships clang without OpenMP support. As such, we have to resort to using GCC.

One (painful) option to build/install GCC:
{% highlight bash %}
$ brew install gcc
# ... and wait for eternity ...
{% endhighlight %}

Fortunately, [HPC on Mac OS X](http://hpc.sourceforge.net/) offers precompiled binaries of GCC.
{% highlight bash %}
$ wget http://downloads.sourceforge.net/project/hpc/hpc/gcc/gcc-5.2-bin.tar.gz
$ tar -xz -C / -f gcc-5.2-bin.tar.gz
# This untars gcc to /usr/local/
{% endhighlight %}

Now we have GCC/OpenMP support and the earlier command works:
{% highlight bash %}
$ THEANO_FLAGS="cxx=g++" python theano/misc/elemwise_openmp_speedup.py
Fast op time without openmp 0.000201s with openmp 0.000162s speedup 1.24
Slow op time without openmp 0.002742s with openmp 0.000902s speedup 3.04
{% endhighlight %}

Happy tuning!
