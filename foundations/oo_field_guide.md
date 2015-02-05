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


