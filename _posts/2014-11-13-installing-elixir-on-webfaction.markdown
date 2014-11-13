---
layout: post
title: "Installing Elixir on WebFaction"
date: 2014-11-13 14:19:00
tags:
  - elixir
---

This is a short guide on installing Elixir on a [WebFaction](https://www.webfaction.com/) shared server. Overall it is fairly easy: first we will install Erlang, then Elixir proper.


Installing Erlang
=================

We will first install erlang in our home directory. The installation itself will be in `$HOME/lib/erlang`, and all binaries will be aliased in `$HOME/bin`.

1. Download the latest erlang release in a temporary folder

{% highlight bash %}
wget http://www.erlang.org/download/otp_src_17.3.tar.gz
{% endhighlight %}

2. Extract the archive

{% highlight bash %}
tar xvzf otp_src_17.3.tar.gz
cd otp_src_17.3
{% endhighlight %}

3. Compile and install erlang

{% highlight bash %}
./configure --prefix=$HOME
make
make install
{% endhighlight %}


Installing Elixir
=================

We will now install elixir in a similar fashion by installing it inside `$HOME/lib/elixir`, and then aliasing all binaries inside `$HOME/bin`.

1. Download the latest elixir release in a temporary folder

{% highlight bash %}
wget http://s3.hex.pm/builds/elixir/v1.0.2.zip
{% endhighlight %}

2. Extract the archive directly in the destination folder

{% highlight bash %}
unzip -d $HOME/lib/elixir v1.0.2.zip
{% endhighlight %}

3. Alias all binaries

{% highlight bash %}
ln -s $HOME/lib/elixir/bin/elixir $HOME/bin/elixir
ln -s $HOME/lib/elixir/bin/elixirc $HOME/bin/elixirc
ln -s $HOME/lib/elixir/bin/iex $HOME/bin/iex
ln -s $HOME/lib/elixir/bin/mix $HOME/bin/mix
{% endhighlight %}
