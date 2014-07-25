---
layout: post
title: "Map to Keyword List"
date: 2014-07-25 16:35:00
tags:
  - elixir
---

I like to write Elixir functions that take a `Keyword List` as optional arguments. Much like for instance [HTTPoison](https://github.com/edgurgel/httpoison) does:

{% highlight elixir %}
def request(method, url, body \\ "", headers \\ [], options \\ []) do
  timeout = Keyword.get options, :timeout, 5000
  stream_to = Keyword.get options, :stream_to
  ...
end
{% endhighlight %}


But sometimes all I have at hand is a `Map`, usually coming straight from a third party json API. Here is a very simple snippet of code to convert such a `Map` into a `Keyword List`:

{% highlight elixir %}
def to_keyword_list(dict) do
  Enum.map(dict, fn({key, value}) -> {String.to_atom(key), value} end)
end
{% endhighlight %}

Of course for this code to work, all keys must be strings!
