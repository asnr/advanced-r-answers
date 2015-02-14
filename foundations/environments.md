Environment basics
------------------

1.  List three ways in which an environment differs from a list.

    1.  The names in an environment must be unique; names in a list can be
        repeated (in which case referencing by name returns the first element with that name).

    2.  There is no notion of ordering in an environment; no name comes before
        another.

    3.  Every environment except the empty environment has a parent; there is
        no equivalent notion for lists.

2.  If you don’t supply an explicit environment, where do `ls()` and `rm()`
    look? Where does `<-` make bindings?

    All 3 work on the current environment, ie. the one returned by environment().

3.  Using `parent.env()` and a loop (or a recursive function), verify that the
    ancestors of `globalenv()` include `baseenv()` and `emptyenv()`. Use the same basic idea to implement your own version of `search()`.

    ```r
    e = globalenv()
    while (TRUE) {
        if (identical(e, baseenv()))
            print('Found the base environment!')
        if (identical(e, emptyenv())) {
            print('Found the empty environment!')
            break
        }
        e = parent.env(e)
    }
    #> [1] "Found the base environment!"
    #> [1] "Found the empty environment!"
    ```

    ```r
    my_search = function() {
        e = globalenv()
        search_path = c()
        while (!identical(e, emptyenv())) {
            search_path = c(search_path, environmentName(e))
            e = parent.env(e)
        }
        return(search_path)
    }
    my_search()
    #> [1] "R_GlobalEnv"       "package:stats"     "package:graphics"
    #> [4] "package:grDevices" "package:utils"     "package:datasets"
    #> [7] "package:methods"   "Autoloads"         "base" 

    # Not quite the same as
    search()
    #> [1] ".GlobalEnv"        "package:stats"     "package:graphics"
    #> [4] "package:grDevices" "package:utils"     "package:datasets"
    #> [7] "package:methods"   "Autoloads"         "package:base"
    # but oh well
    ```


Recursing over environments
---------------------------


Function environments
---------------------