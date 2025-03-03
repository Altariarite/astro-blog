---
title: 'Working through the book ANSI Common Lisp in Clojurescript - Macros'
pubDate: 2025-03-03
description: 'A post about Macros'
author: 'Clojure Learner'
image:
    url: 'https://docs.astro.build/assets/rose.webp'
    alt: 'The Astro logo on a dark background with a pink glow.'
tags: ["astro", "blogging", "learning in public", "clojure"]
---

```clojure
(def i (atom 1))
;;=> #'mini.playground/x
@i
;;=> 1

(defmacro nil! [x]
  (list 'reset! x nil))

(nil! i)
;;=> nil
@i
;;=> nil

;; to test a macro we look at it's expansion
(macroexpand-1 '(nil! x))
;;=> (reset! x nil)

`(a b c)

(def a 1)
(def b 2)
`(a is ~a and b is ~b)

(defmacro nil! [x]
  `(reset! ~x nil))

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