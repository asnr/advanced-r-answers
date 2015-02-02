Quiz
----

1.  What are the three components of a function?

    `body`, `environment`, `formals`
    
2.  What does the following code return?
    ```r
    x <- 10
    f1 <- function(x) {
      function() {
        x + 10
      }
    }
    f1(1)()
    ```
    
    `11`.

3.  How would you more typically write this code?
    ```r
    `+`(1, `*`(2, 3))
    ```
    
    ```r
    1 + (2*3)
    ```

4.  How could you make this call easier to read?
    ```r
    mean(, TRUE, x = c(1:10, NA))
    ```
    
    ```r
    mean(c(1:10, NA), na.rm=TRUE)
    ```

5.  Does the following function throw an error when called? Why/why not?
    ```r
    f2 <- function(a, b) {
      a * 10
    }
    f2(10, stop("This is an error!"))
    ```
    
    No, it doesn't throw an error because arguments are evaluated lazily and `b` 
    is never evaluated in the function body.

6.  What is an infix function? How do you write it? What’s a replacement function? How do you write it?

    An infix function is a binary function that can be written as a binary operator, eg. ``1+2 == `+`(1, 2)``.
    To define an infix function, wrap the name in `%` and then backticks:
    ```r
    `%inf%` = function(a, b) {
        ...
    }
    c = a %inf% b
    ```
    
    A replacement function is syntactic sugar that lets you present an interface as
    if the function modified its first arguments in place. To declare a replacement function, end its name
    with `<-`, enclose the name in backticks, give it an argument called `value` and remember that
    because this syntax doesn't provide a means to actually modify an object in place, you need to return
    the new object:
    ```r
    `first<-` = function(x, value) {
        x[1] = value
        x
    }
    y = rep(3, 3)
    first(y) = 1
    y
    # [1] 1 3 3
    ```
    
7.  What function do you use to ensure that a cleanup action occurs regardless of how a function terminates?

    `on.exit()`


Function Components
-------------------

1.  What function allows you to tell if an object is a function?
    What function allows you to tell if a function is a primitive function?

    `is.function()` and `is.primitive()`

2.  This code makes a list of all functions in the base package.
    ```r
    objs <- mget(ls("package:base"), inherits = TRUE)
    funs <- Filter(is.function, objs)
    ```

   Use it to answer the following questions:

   1.  Which base function has the most arguments?

       The `funs` array holds objects which in turn store a function as an attribute. To unwrap this mess we need

        ```r
        extract_fun = function(fun_obj) fun_obj[[ names(fun_obj)[1] ]]
        ```

        Counting the number of arguments is also a bit delicate because the list contains both closures and primitives, hence

        ```r
        fun_obj_args_len = function(fun_obj) { length(formals(args(extract_fun(fun_obj)))) }
        ```

        We can now loop over the list and find the function with the most arguments:

        ```r
        lengths = 1:length(funs)
        for (i in 1:length(funs)) {  # SUBMIT A STACKOVERFLOW QUESTION - WHY DOESN'T SAPPLY WORK HERE?
            lengths[i] = fun_obj_args_len(funs[i])
        }
        funs[which.max(lengths)]  # scan function, 22 arguments
        ```

   2. How many base functions have no arguments?
      What’s special about those functions?

      ```r
      no_args = lengths == 0
      funs[no_args]
      ```

      70 functions have no arguments. No idea what's special about them. There are infix operators like `[` and `@` as well as OS calls like `Sys.time()` and random function like `contributors()` which lists contributors to the R project.

    3. How could you adapt the code to find all primitive functions?

       ```r
       objs <- mget(ls("package:base"), inherits = TRUE)
       funs <- Filter(is.primitive, objs)
       ```

3.  What are the three important components of a function?

    `formals`, `body`, `environment`

4.  When does printing a function not show what environment it was created in?

    When the environment is the global environment.


Every Operation Is A Function Call
----------------------------------

1.  Clarify the following list of odd function calls:
    ```r
    x <- sample(replace = TRUE, 20, x = c(1:10, NA))
    y <- runif(min = 0, max = 1, 20)
    cor(m = "k", y = y, u = "p", x = x)
    ```

    They are equivalent to
    ```r
    x <- sample(c(1:10, NA), 20, replace=TRUE)  # sample {1, 2, ..., 10, NA} 20 times w/ replacement
    y <- runif(20, min=0, max=1)  # randomly sample X~uniform(0, 1)
    cor(x, y, method="kendall", use="pairwise.complete.obs")
    ```

2.  What does this function return? Why? Which principle does it illustrate?
    ```r
    f1 <- function(x = {y <- 1; 2}, y = 0) {
      x + y
    }
    f1()
    ```

    `3`. `x` is evaluated before `y`, which causes the original promise of `y` to be overwritten by `1`. Compare with
    ```r
    g2 <- function(x = {y <- 1; 2}, y = x) { y + x }
    g2()  # 4
    g3 <- function(x = {y <- 1; 2}, y = 0)  { y + x }
    g3()  # 2
    ```

3.  What does this function return? Why? Which principle does it illustrate?
    ```r
    f2 <- function(x = z) {
      z <- 100
      x
    }
    f2()
    ```

    `100`. Illustrates lazy evaluation.

Special Calls
-------------

1.  Create a list of all the replacement functions found in the base package.
    Which ones are primitive functions?

    ```r
    library(magrittr)
    library(stringr)
    is_replacement = . %>% str_detect(., '<-$')
    objs <- mget(ls("package:base"), inherits = TRUE)
    repl_funs = Filter(is_replacement, names(objs))
    repl_funs
     [1] "[[<-"             "[<-"              "@<-"              "<-"
     [5] "<<-"              "$<-"              "attr<-"           "attributes<-"
     [9] "body<-"           "class<-"          "colnames<-"       "comment<-"
    [13] "diag<-"           "dim<-"            "dimnames<-"       "Encoding<-"
    [17] "environment<-"    "formals<-"        "is.na<-"          "length<-"
    [21] "levels<-"         "mode<-"           "mostattributes<-" "names<-"
    [25] "oldClass<-"       "parent.env<-"     "regmatches<-"     "row.names<-"
    [29] "rownames<-"       "split<-"          "storage.mode<-"   "substr<-"
    [33] "substring<-"      "units<-"
    length(repl_funs)
    [1] 34

    # Primitive replacement functions:
    prim_repl_funs = Filter(is.primitive, objs[repl_funs])
    names(prim_repl_funs)
     [1] "[[<-"           "[<-"            "@<-"            "<-"
     [5] "<<-"            "$<-"            "attr<-"         "attributes<-"
     [9] "class<-"        "dim<-"          "dimnames<-"     "environment<-"
    [13] "length<-"       "levels<-"       "names<-"        "oldClass<-"
    [17] "storage.mode<-"
    ```

2.  What are valid names for user-created infix functions?

    Not the dumb ones. Don't use `%`.

3.  Create an infix `xor()` operator.

    ```r
    `%xor%` = function(a, b) (a && !b) || (!a && b)
    ```

4.  Create infix versions of the set functions `intersect()`, `union()`,
    and `setdiff()`.

    ```r
    `%intersect%` = function(...) intersect(...)
    `%union%`     = function(...) union(...)
    `%setdiff%`   = function(...) setdiff(...)
    ```

5.  Create a replacement function that modifies a random location in a vector.

    ```r
    # Note that the function needs to return 'x', otherwise we will just
    # assign 'value'!
    `randmod<-` = function(x, value) { x[sample(1:length(x), 1)] = value; x }
    y = 1:10
    randmod(y) <- -1
    y
     [1]  1  2  3 -1  5  6  7  8  9 10
    ```


Return Values
-------------
