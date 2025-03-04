---
layout: ../../layouts/MarkdownPostLayout.astro
title: 'Macros in Clojure'
pubDate: 2025-03-03
description: 'A post about Macros'
author: 'Clojure Learner'
image:
    url: 'https://docs.astro.build/assets/rose.webp'
    alt: 'The Astro logo on a dark background with a pink glow.'
tags: ["astro", "blogging", "learning in public", "clojure"]
---
I tried to take part of the *Macros* chapter from Paul Graham's *ANSI Common Lisp* and adapted it to Clojure. The code blocks are powered by Klipse and interactive.
## Setup
Some quirks of the online repl means we need to switch to a special namespace
```klipse
(ns my.m$macros) ;; define macros in the namespace my.m
```
## Intro to Macros

A `defmacro` looks a lot like a `defun`, but instead of defining the value a call should produce, it defines how a call should be translated.

```klipse
(defmacro nil! [x]
  (list 'reset! x nil))
```

```klipse
(def i (atom 1))
@i
```

```klipse
(my.m/nil! i)
@i
```

To test a macro we look at it's expansion
```klipse
(macroexpand-1 '(my.m/nil! x))
```

TODO: adapt the function part

## Backtick `
[^1]: Common Lisp actually use backquote instead of backtick. Clojure use a different set of symbols. Similarly ~ is , and ~@ is ,@ in Common Lisp.

The backtick read-macro makes it possible to build lists from templates. Used by itself, it looks like a normal quote, but resolves to fully-qualified symbols.
```klipse
'(a b c)
```
```klipse
`(a b c)
```
The advantage of backtick is that, within a backtick expression, you can use `~` (tilda, or unquote) and `~@` (tilda-at, or "unquote splicing") to turn evaluation back on[^1].
```klipse
(def a 1)
(def b 2)
`(a is ~a and b is ~b)
```

By using backquote instead of a call to `list`, we can write macro definitions that look like the expansions they will produce. 
```klipse
(defmacro nil! [x]
  `(reset! ~x nil))
```
```
(def lst '(a b c))

`(lst is ~@lst)

(dotimes [_ 10] (print "."))

(p/pprint
 (macroexpand-1 '(cond (a) b
                       (c) (d e)
                       :else f)))

;;  correct ntimes in clojure
(comment
  ;; appending # to the symbol generates unique symbol names
  `(a#)
  ;;=> (a__10521__auto__)


  :rcf)

(defmacro ntimes [n & body]
  `(loop [i# 0]
     (when (< i# ~n)
       ~@body
       (recur (inc i#)))))

(macroexpand-1 (ntimes 10 (print ".")))

(ntimes 10 (print "."))
```

