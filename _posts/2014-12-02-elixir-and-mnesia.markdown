---
layout: post
title: "Elixir and (A)mnesia"
date: 2014-12-02 16:55:00
tags:
  - elixir
---


I've recently written a small website based on [Phoenix](https://github.com/phoenixframework/phoenix) and [Amnesia](https://github.com/meh/amnesia). Amnesia is an Elixir wrapper around the built-in [Mnesia](http://www.erlang.org/doc/apps/mnesia/Mnesia_chap1.html) database.


Specifying the mnesia dir from the the shell can be done easily this way:

{% highlight bash %}
elixir --erl "-mnesia dir 'path/to/dir'" -S mix phoenix.start
{% endhighlight %}
