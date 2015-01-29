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
    f2 <- function(x = { y <- 1; 2}, y = x) { y + x }
    f2()  # 4
    f3 <- function(x = {y <- 1; 2}, y = 0)  { y + x }
    f3()  # 2
    ```


Special Calls
-------------
