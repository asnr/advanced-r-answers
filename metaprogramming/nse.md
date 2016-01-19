Capturing expressions
---------------------

 1. Let `g = function(x) deparse(substitute(x))`. Why does

    ```r
    g(a + b + c + d + e + f + g + h + i + j + k + l + m +
      n + o + p + q + r + s + t + u + v + w + x + y + z)
    ```

    return a character vector of length 2? Write a wrapper around `deparse()` so that it always returns a single string.

    The `deparse` function takes a `width.cutoff` argument, which takes integer values in `[20, 500]`. `deparse` will start creating a new character element if it has reached the end of a symbol and has already printed at least `width.cutoff` bytes into the current character element.

    ```r
    g2 = function(x) paste(deparse(substitute(x)), collapse = " ")
    ```

 2. Why does `as.Date.default()` use `substitute()` and `deparse()`? Why does
    `pairwise.t.test()` use them?

    `as.Date.default()` uses `substitute()` and `deparse()` for printing an error message when the argument can't be converted to `Date`:

    ```r
    stop(gettextf("do not know how to convert '%s' to class %s", 
         deparse(substitute(x)), dQuote("Date")), domain = NA)
    ```

    `pairwise.t.test()` uses `substitute()` and `deparse()` to print out the names of the columns being tested.


 3. `pairwise.t.test()` assumes that `deparse()` always returns a length one
    character vector. Can you construct an input that violates this expectation?

    need something like `pairwise.t.test(c(aoriestnaoirsetn, oairesntaoriesntaorsiteanr))` --- needs to have a space in there that's not part of symbol name.