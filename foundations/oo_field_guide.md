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
    
    


