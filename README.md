# The Exercism Clojure Representer

What is a representer? You can read about them in [this article](https://exercism.org/blog/introducing-representers), or just watch Jeremy explain it in this video:

[![Representer video](https://img.youtube.com/vi/OJqN9adA_6Y/0.jpg)](http://www.youtube.com/embed/OJqN9adA_6Y)

The idea is very simple: We receive hundreds of exercise submissions each week, many of which are quite similar. In order to produce the most valuable feedback we can from our mentors, we have a piece of software that analyzes the solutions and identifies *common approaches* used. It does this by normalizing for various things, namely whitespace, macroexpansion, and variable names.

## Clojure Representer implementation

Most of the work is done using `clojure.tools.analyzer`, with additional processing done with `rewrite-clj`. We make use of the analyzer for its macroexpansion capability, as well as its ability to emit Clojure forms from the generated AST. Specifically, we use the `emit-hygienic-form` function which includes a pass called `uniquify` which normally replaces all the local variables with unique identifiers, but use a local version that is modified slightly so that instead of making the names *unique*, it makes them *generic*. This normalizes the solutions for variable names, replacing each occurrance with a placeholder name.

In a future post we will dive more into the implementation, but that's the basic idea. Here, what I'd like to do is just show it in action by demonstrating its practical use - grouping the solutions into common approaches.

## Common approaches to the `two-fer` exercise

I ran the last 500 submissions to `two-fer` through the representer and processed them into a map using this function:

``` clojure
(defn representations [exercise]
  (let [representations
        (apply merge
               (for [n (solution-dirs exercise)]
                 {n (representation exercise n)}))
        repmap-seq      (for [n (solution-dirs exercise)]
                          {:solution       n
                           :code           (solution exercise n)
                           :representation (get representations n)
                           :times-used     (get (frequencies (vals representations))
                                                (representation exercise n))})]
    (reverse (sort-by :times-used
                      (map #(select-keys % [:code :times-used])
                           (distinct-by :representation repmap-seq))))))
```

This creates a sequence of maps consisting of only the solutions resulting in unique representations, and sorts them by the number of times used, highest to lowest.

### Approach 1 - 114 submissions

``` clojure
(defn two-fer 
  ([] (two-fer "you")) 
  ([name] (str "One for " name ", one for me.")))
```

It's a typical multi-arity function, where the nullary arity calls the unary arity supplying the default argument.

### Approach 2 - 79 submissions

``` clojure
(defn two-fer 
  ([] "One for you, one for me.") 
  ([name] (str "One for " name ", one for me.")))
```

Here instead of calling itself, the default result is supplied as its own string.

### Approach 3 - 37 submissions

``` clojure
(defn two-fer 
  ([] (two-fer "you")) 
  ([name] (format "One for %s, one for me." name)))
```

It is like the first approach, but the variable is interpolated using `format` instead of concatenated with `str`.

### Approach 4 - 29 submissions

``` clojure
(defn two-fer 
  ([] (str "One for you, one for me.")) 
  ([name] (str "One for " name ", one for me.")))
```

Lol, they're concatenating a single string... since this is functionally identical to the second approach, so it might be worth adding this to the representer as a rule.

### Approach 5 - 25 submissions

``` clojure
(defn two-fer 
  ([] "One for you, one for me.") 
  ([name] (format "One for %s, one for me." name)))
```

This is like the second approach, but using `format` instead of `str`.

### Approach 6 - 24 submissions

``` clojure
(defn two-fer 
  ([name] (str "One for " name ", one for me.")) 
  ([] (two-fer "you")))
```

This is like the first approach, but with the arities switched! This is another way we could improve the representer, by having it sort them.

### Approach 7 - 23 submissions

``` clojure
(defn two-fer 
  ([name] (str "One for " name ", one for me.")) 
  ([] "One for you, one for me."))
```

See what's going on here?

### Approach 8 - 11 submissions

``` clojure
(defn two-fer 
  ([name] (str "One for " name ", one for me.")) 
  ([] (str "One for you, one for me.")))
```

Yep. It's like permutations of ice cream flavors!

### Approach 9 - 8 submissions

``` clojure
(defn two-fer 
  ([name] (format "One for %s, one for me." name)) 
  ([] "One for you, one for me."))
```

Build string with `format`; arities reversed

### Approach 10 - 5 submissions

``` clojure
(defn- _two-fer [name] 
  (str "One for " name ", one for me."))
   
(defn two-fer 
  ([] (_two-fer "you")) 
  ([name] (_two-fer name)))
```

This is a new one - we have a helper function that builds the string, allowing the main function to be simplified down to a dispatch function. While this may not seem like a huge advantage here, it is a useful pattern to understand for when dealing with more complex functions later.

### Approach 11 - 4 submissions

``` clojure
(defn two-fer [& [name]] 
  (format "One for %s, one for me." (or name "you")))
```

This one uses a variadic function, taking a variable number of arguments instead of writing a separate function body for each arity. It uses `or` to return `name` is it is non-nil, otherwise "you".

Approach 12 - 3 submissions

``` clojure
(defn two-fer 
  ([name] (format "One for %s, one for me." name)) 
  ([] (two-fer "you")))
```

Format, reversed function bodies, one arity calls the other.

### Approach 13 - 3 submissions

``` clojure
(defn two-fer [& name] 
  (str "One for " (or (first name) "you") ", one for me."))
```

Like the variadic solution above, but building it with `str` instead of `format`, and the argument list is slightly different in that it isn't destructured.
