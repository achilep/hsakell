# xeno

[![Github actions build status](https://img.shields.io/github/workflow/status/ocramz/xeno/Stack)](https://github.com/ocramz/xeno/actions) [![Hackage version](https://img.shields.io/hackage/v/xeno.svg?label=Hackage)](https://hackage.haskell.org/package/xeno) [![Stackage version](https://www.stackage.org/package/xeno/badge/lts?label=Stackage)](https://www.stackage.org/package/xeno)


A fast event-based XML parser.

[Blog post](http://chrisdone.com/posts/fast-haskell-c-parsing-xml).

## Features

* SAX-style/fold parser which triggers events for open/close
  tags, attributes, text, etc.
* Low memory use (see memory benchmarks below).
* Very fast (see speed benchmarks below).
* It
  [cheats like Hexml does](http://neilmitchell.blogspot.co.uk/2016/12/new-xml-parser-hexml.html)
  (doesn't expand entities, or most of the XML standard).
* Written in pure Haskell.
* CDATA is supported as of version 0.2.

Please see the bottom of this file for guidelines on contributing to this library.


## Performance goals

The [hexml](https://github.com/ndmitchell/hexml) Haskell library uses
an XML parser written in C, so that is the baseline we're trying to
beat or match roughly.

![Imgur](http://i.imgur.com/XgdZoQ9.png)

The `Xeno.SAX` module is faster than Hexml for simply walking the
document. Hexml actually does more work, allocating a DOM. `Xeno.DOM`
is slighly slower or faster than Hexml depending on the document,
although it is 2x slower on a 211KB document.

Memory benchmarks for Xeno:

    Case                Bytes  GCs  Check
    4kb/xeno/sax        2,376    0  OK
    31kb/xeno/sax       1,824    0  OK
    211kb/xeno/sax     56,832    0  OK
    4kb/xeno/dom       11,360    0  OK
    31kb/xeno/dom      10,352    0  OK
    211kb/xeno/dom  1,082,816    0  OK

I memory benchmarked Hexml, but most of its allocation happens in C,
which GHC doesn't track. So the data wasn't useful to compare.

Speed benchmarks:

    benchmarking 4KB/hexml/dom
    time                 6.317 ??s   (6.279 ??s .. 6.354 ??s)
                         1.000 R??   (1.000 R?? .. 1.000 R??)
    mean                 6.333 ??s   (6.307 ??s .. 6.362 ??s)
    std dev              97.15 ns   (77.15 ns .. 125.3 ns)
    variance introduced by outliers: 13% (moderately inflated)

    benchmarking 4KB/xeno/sax
    time                 5.152 ??s   (5.131 ??s .. 5.179 ??s)
                         1.000 R??   (1.000 R?? .. 1.000 R??)
    mean                 5.139 ??s   (5.128 ??s .. 5.161 ??s)
    std dev              58.02 ns   (41.25 ns .. 85.41 ns)

    benchmarking 4KB/xeno/dom
    time                 10.93 ??s   (10.83 ??s .. 11.14 ??s)
                         0.994 R??   (0.983 R?? .. 0.999 R??)
    mean                 11.35 ??s   (11.12 ??s .. 11.91 ??s)
    std dev              1.188 ??s   (458.7 ns .. 2.148 ??s)
    variance introduced by outliers: 87% (severely inflated)

    benchmarking 31KB/hexml/dom
    time                 9.405 ??s   (9.348 ??s .. 9.480 ??s)
                         0.999 R??   (0.998 R?? .. 0.999 R??)
    mean                 9.745 ??s   (9.599 ??s .. 10.06 ??s)
    std dev              745.3 ns   (598.6 ns .. 902.4 ns)
    variance introduced by outliers: 78% (severely inflated)

    benchmarking 31KB/xeno/sax
    time                 2.736 ??s   (2.723 ??s .. 2.753 ??s)
                         1.000 R??   (1.000 R?? .. 1.000 R??)
    mean                 2.757 ??s   (2.742 ??s .. 2.791 ??s)
    std dev              76.93 ns   (43.62 ns .. 136.1 ns)
    variance introduced by outliers: 35% (moderately inflated)

    benchmarking 31KB/xeno/dom
    time                 5.767 ??s   (5.735 ??s .. 5.814 ??s)
                         0.999 R??   (0.999 R?? .. 1.000 R??)
    mean                 5.759 ??s   (5.728 ??s .. 5.810 ??s)
    std dev              127.3 ns   (79.02 ns .. 177.2 ns)
    variance introduced by outliers: 24% (moderately inflated)

    benchmarking 211KB/hexml/dom
    time                 260.3 ??s   (259.8 ??s .. 260.8 ??s)
                         1.000 R??   (1.000 R?? .. 1.000 R??)
    mean                 259.9 ??s   (259.7 ??s .. 260.3 ??s)
    std dev              959.9 ns   (821.8 ns .. 1.178 ??s)

    benchmarking 211KB/xeno/sax
    time                 249.2 ??s   (248.5 ??s .. 250.1 ??s)
                         1.000 R??   (1.000 R?? .. 1.000 R??)
    mean                 251.5 ??s   (250.6 ??s .. 253.0 ??s)
    std dev              3.944 ??s   (3.032 ??s .. 5.345 ??s)

    benchmarking 211KB/xeno/dom
    time                 543.1 ??s   (539.4 ??s .. 547.0 ??s)
                         0.999 R??   (0.999 R?? .. 1.000 R??)
    mean                 550.0 ??s   (545.3 ??s .. 553.6 ??s)
    std dev              14.39 ??s   (12.45 ??s .. 17.12 ??s)
    variance introduced by outliers: 17% (moderately inflated)

## DOM Example

Easy as running the parse function:

``` haskell
> parse "<p key='val' x=\"foo\" k=\"\"><a><hr/>hi</a><b>sup</b>hi</p>"
Right
  (Node
     "p"
     [("key", "val"), ("x", "foo"), ("k", "")]
     [ Element (Node "a" [] [Element (Node "hr" [] []), Text "hi"])
     , Element (Node "b" [] [Text "sup"])
     , Text "hi"
     ])
```

## SAX Example

Quickly dumping XML:

``` haskell
> let input = "Text<tag prop='value'>Hello, World!</tag><x><y prop=\"x\">Content!</y></x>Trailing."
> dump input
"Text"
<tag prop="value">
  "Hello, World!"
</tag>
<x>
  <y prop="x">
    "Content!"
  </y>
</x>
"Trailing."
```

Folding over XML:

``` haskell
> fold const (\m _ _ -> m + 1) const const const const 0 input -- Count attributes.
Right 2
```

``` haskell
> fold (\m _ -> m + 1) (\m _ _ -> m) const const const const 0 input -- Count elements.
Right 3
```

Most general XML processor:

``` haskell
process
  :: Monad m
  => (ByteString -> m ())               -- ^ Open tag.
  -> (ByteString -> ByteString -> m ()) -- ^ Tag attribute.
  -> (ByteString -> m ())               -- ^ End open tag.
  -> (ByteString -> m ())               -- ^ Text.
  -> (ByteString -> m ())               -- ^ Close tag.
  -> ByteString                         -- ^ Input string.
  -> m ()
```

You can use any monad you want. IO, State, etc. For example, `fold` is
implemented like this:

``` haskell
fold openF attrF endOpenF textF closeF s str =
  execState
    (process
       (\name -> modify (\s' -> openF s' name))
       (\key value -> modify (\s' -> attrF s' key value))
       (\name -> modify (\s' -> endOpenF s' name))
       (\text -> modify (\s' -> textF s' text))
       (\name -> modify (\s' -> closeF s' name))
       str)
    s
```

The `process` is marked as INLINE, which means use-sites of it will
inline, and your particular monad's type will be potentially erased
for great performance.


## Contributors

See CONTRIBUTORS.md


## Contribution guidelines

All contributions and bug fixes are welcome and will be credited appropriately, as long as they are aligned with the goals of this library: speed and memory efficiency. In practical terms, patches and additional features should not introduce significant performance regressions.
