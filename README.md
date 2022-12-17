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

