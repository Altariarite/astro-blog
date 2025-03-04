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
Lisp macros are what makes it so unique and powerful. It finally clicked for me when I was reading Paul Graham's [*ANSI Common Lisp*](https://paulgraham.com/acl.html). So I tried to take part of the *Macros* chapter from the book and adapted it to Clojure. 

The code blocks are powered by Klipse and interactive.

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
[^1]: Common Lisp uses backquote instead of backtick. Clojure uses a different set of symbols. Similarly `~` is `,` and `~@` is `,@` in Common Lisp.

The backtick read-macro makes it possible to build lists from templates. Used by itself, it looks like a normal quote, but resolves to namespace-qualified symbols.
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

Example using `~@`
```klipse
(def lst '(a b c))

`(lst is ~@lst)
```

By using backquote instead of a call to `list`, we can write macro definitions that look like the expansions they will produce. 
```klipse
(defmacro nil2! [x]
  `(reset! ~x nil))
```

`nil2!` is equivalent to `nil!`
```klipse
(macroexpand-1 '(my.m/nil2! x))
```



We are now able to understand the `cond` macro.
```klipse
 (macroexpand-1 '(cond (a) b
                       (c) (d e)
                       :else f))
```

[The ClojureScript API](https://cljs.github.io/api/cljs.core/cond) page includes the source code for `cond`. Can you match the source code to the macro expansion?

```clojure
(defmacro cond
  {:added "1.0"}
  [& clauses]
    (when clauses
      (list 'if (first clauses)
            (if (next clauses)
                (second clauses)
                (throw (IllegalArgumentException.
                         "cond requires an even number of forms")))
            (cons 'clojure.core/cond (next (next clauses))))))
```

## Correct ntimes
We want to write a macro `ntimes`. When calling `(ntimes 10 (print "."))` it achieves the same effect as the following `dotimes`
```klipse
(dotimes [_ 10] (print ".")) ;; print . 10 times
```
In the book Paul Graham talked about gotchas when writing Macros in Common Lisp. Many gotchas don't apply to clojure. However there is one common problem that needs to be addressed.

Let's first look at an incorrect version of `ntimes`:
```klipse
(defmacro ntimes [n & body]
  `(loop [i 0]
     (when (< i ~n)
       ~@body
       (recur (inc i)))))
```
```klipse
(my.m/ntimes 10 (print "."))
```

Oops, that didn't work. What's wrong? [Turns out](https://clojure-doc.org/articles/language/macros/) writing macro is dangerous, because the `i` in macro might shadow some other `i` outside the scope of the macro. If this is confusing, don't worry as Clojure is being very nice and already thought about this, and that's why `` ` ``(backtick) resolves to namespace-qualified symbols.

```klipse
(macroexpand-1 '(my.m/ntimes 10 (print ".")))
```
In this case, the `i` in the loop resolves to `my.m/i`, and as you can probably tell from the error message, you cannot have namespace-qualified symbols as a local name.

To write the macro correctly, we need another tool - symbol generation. 

We can generate unique symbol names by appending `#` to the symbol.

```klipse
`(a#)
```

```klipse
(defmacro ntimes [n & body]
  `(loop [i# 0]
     (when (< i# ~n)
       ~@body
       (recur (inc i#)))))
```
Now the symbol is properly named and local, and there's very little chance that it will overlap with the same symbol name outside the macro.
```klipse
(macroexpand-1 '(my.m/ntimes 10 (print ".")))
```
Let's test our macro:

```klipse
(my.m/ntimes 10 (print "."))
```
![noice!](https://media.giphy.com/media/v1.Y2lkPTc5MGI3NjExbnE1bmw4d2FneWx5aXR4ZzlteHFlZTNkYWhxZDM0bmJ4cTFlZGdjaCZlcD12MV9naWZzX3NlYXJjaCZjdD1n/yJFeycRK2DB4c/giphy.gif)