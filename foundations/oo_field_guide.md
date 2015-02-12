S3
--

1.  Read the source code for `t()` and `t.test()` and confirm that `t.test()` is an S3 generic and not an S3 method.
    What happens if you create an object with class `test` and call `t()` with it?
    
    ```r
    body(t)
    #> UseMethod("t")
    body(t.test)
    #> UseMethod("t.test")
    
    x = list()
    class(x) = 'test'
    t(x)
    #> Error: is.atomic(x) is not TRUE
    #> In addition: Warning message:
    #> In mean.default(x) : argument is not numeric or logical: returning NA
    ```
    
    This is error message is not terribly useful in determining how `t(x)` is dispatched. Instead do
    
    ```r
    y = c(100, 200)
    class(y) = 'test'
    t(y)  # output is identical to t.test(y)
    
    # but
    z = c(100, 200)
    t(z)
    #>      [,1] [,2]
    #> [1,]  100  200
    ```
    
    from which we conclude that `t(y)` is being dispatched to `t.test(y)` which does its own dispatching in turn.
    
    This is clearly undesirable behaviour. To avoid it, we shouldn't use periods `.` in function names; if this
    had been the case, then `t_test(y)` would not be called from `t(y)`.
    
2.  What classes have a method for the `Math` group generic in base R?
    Read the source code. How do the methods work?
    
    ```r
    methods(Math)
    #> [1] Math.data.frame Math.Date       Math.difftime   Math.factor     Math.POSIXt
    ```
    
    Source code:
    ```r
    body(Math.data.frame)
    #> {
    #>    mode.ok <- vapply(x, function(x) is.numeric(x) || is.complex(x), 
    #>        NA)
    #>    if (all(mode.ok)) {
    #>        x[] <- lapply(X = x, FUN = .Generic, ...)
    #>        return(x)
    #>    }
    #>    else {
    #>        vnames <- names(x)
    #>        if (is.null(vnames)) 
    #>            vnames <- seq_along(x)
    #>        stop("non-numeric variable in data frame: ", vnames[!mode.ok])
    #>    }
    #> }
    ```
    This function checks if all of the columns are numeric or complex and if they are it then `lapply`s the
    relevant `Math` function held in `.Generic` (eg. `abs`, `sqrt`) along the columns. If any of the columns
    aren't complex or numeric it lets you know. **I don't know** why the third argument to `vapply` is `NA` nor
    why the result of `lapply` is set to `x[]` and not `x`.

    ```r
    body(Math.difftime)
    #> {
    #>     switch(.Generic, abs = , sign = , floor = , ceiling = , trunc = , 
    #>         round = , signif = {
    #>             units <- attr(x, "units")
    #>             .difftime(NextMethod(), units)
    #>         }, stop(gettextf("'%s' not defined for \"difftime\" objects", 
    #>             .Generic), domain = NA))
    #> }
    ```
    Throws error if the `.Generic` function isn't one of `abs`, `sign`, `floor`, `ceiling`, `trunc`, `round`
    or `signif`. If it is one of those it evaluates as per normal and for `signif` there is extra handling
    which does **I don't know**.
    
    
    ```r
    body(Math.factor)
    #> stop(sQuote(.Generic), " not meaningful for factors")
    
    body(Math.Date)
    #> stop(gettextf("%s not defined for \"Date\" objects", .Generic), 
    #>      domain = NA)
    
    body(Math.POSIXt)
    #> {
    #>     stop(gettextf("'%s' not defined for \"POSIXt\" objects", 
    #>         .Generic), domain = NA)
    #> }
    ```
    Just throw error.

3.  R has two classes for representing date time data, POSIXct and POSIXlt, which both inherit from POSIXt.
    Which generics have different behaviours for the two classes? Which generics share the same behaviour?

    POSIXct is epoch time, ie. seconds since the beginning of 1970. Traditionally stored as a 32-bit signed
    `int`, but R stores it internally as a `double`. POSIXlt is a tuple of integers representing year, month,
    day of the month, hour, minute and second.
    See the [original paper](http://cran.r-project.org/doc/Rnews/Rnews_2001-2.pdf#chapter*.12)
    describing their implementation of the R internals more information.

4.  Which base generic has the greatest number of defined methods?

    ```r
    objs <- mget(ls("package:base"), inherits = TRUE)

    is_s3_generic = function(f) all(c("generic", "s3") %in% ftype(f))
    is_generic = function(f) "generic" %in% ftype(f)
    is_not_internal  = function(f) !("internal"  %in% ftype(f))
    is_not_primitive = function(f) !("primitive" %in% ftype(f))

    s3_gens = Filter(is_s3_generic, objs)
    length(s3_gens)  # 65
    gens = Filter(is_generic, objs)
    length(gens)  # 177
    gens_not_internal = Filter(is_not_internal, gens)
    length(gens_not_internal)  # 166
    gens_not_int_nor_prim = Filter(is_not_primitive, gens_not_internal)
    length(gens_not_int_nor_prim)  # 67

    max_idx_not_int_nor_prim = which.max(lapply(names(gens_not_int_nor_prim),
                                                num_methods))
    gens_not_int_nor_prim[max_idx_not_int_nor_prim]
    #> $print
    #> ...

    max_idx_s3_gens = which.max(lapply(names(s3_gens), num_methods))
    s3_gens[max_idx_s3_gens]
    #> $print
    #> ...
    ```

5.  UseMethod() calls methods in a special way. Predict what the following
    code will return, then run it and read the help for UseMethod() to figure
    out what’s going on. Write down the rules in the simplest form possible.
    ```r
    y <- 1
    g <- function(x) {
      y <- 2
      UseMethod("g")
    }
    g.numeric <- function(x) y
    g(10)

    h <- function(x) {
      x <- 10
      UseMethod("h")
    }
    h.character <- function(x) paste("char", x)
    h.numeric <- function(x) paste("num", x)

    h("a")
    ```

    ```r
    g(10)
    #> 2

    h("a")
    #> "char a"
    ```
    
    From the man page, `UseMethod` does the following:

    1.  Find the context for the calling function (the generic): this
        gives us the unevaluated arguments for the original call.

    2.  Evaluate the object (usually an argument) to be used for
        dispatch, and find a method (possibly the default method) or
        throw an error.

    3.  Create an environment for evaluating the method and insert
        special variables (see below) into that environment.  Also
        copy any variables in the environment of the generic that are
        not formal (or actual) arguments.

    4.  Fix up the argument list to be the arguments of the call
        matched to the formals of the method.

    So when `g(10)` is called, `y` is copied into the environment of the
    dispatchee. Similarly, when `h("a")` is called, `x` with the value of
    `10` is copied to the environment of the dispatchee but it is then clobbered by the actual argument passed to the dispatcher.

    Interestingly, if no object is specified to `UseMethod` (second argument,
    see the man page), then the first argument to the generic function is
    evaluated to determine the correct dispatchee, ie. this triggers
    evaluation before you would expect from a lazily-evaluated argument:
    ```r
    f = function() {print('hi'); return(1); }

    g = function(x)    UseMethod('g')
    h = function(x, y) UseMethod('h')

    g.numeric = function(x)    deparse(substitute(x))
    h.numeric = function(x, y) deparse(substitute(y))

    g(f())
    #> [1] "hi"
    #> [1] "f()"
    h(1, f())
    #> [1] "f()"
    ```
    **COME BACK AND CHECK: CAN I TRIGGER EVALUATION TWICE LIKE THIS, EVEN
    WHEN THE ARGUMENT IS ONLY REFERENCED ONCE? PROBABLY NOT---ONCE EVALUATED
    BY THE GENERIC, THE THUNK IS PROBABLY REPLACED BY THE EVALUATED VALUE**


