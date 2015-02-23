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

1.  Create a recursive function to find all environments that contain a
    binding for name.

    ```r
    envs_with = function(name, env=parent.frame()) {
        
        if (identical(env, emptyenv()))
            env_name_vec = NULL
        else if (exists(name, envir=env, inherits=FALSE))
            env_name_vec = c(env,
                             envs_with(name, parent.env(env)))
        else
            env_name_vec = envs_with(name, parent.env(env))

        return(env_name_vec)
    }
    
    unlist(lapply(envs_with('mean'), environmentName))
    #> [1] "base"

    mean = 1
    unlist(lapply(envs_with('mean'), environmentName))
    #> [1] "R_GlobalEnv" "base"
    ```

2.  Write your own version of `get()` using a function written in the style of
    `where()`.

    ```r
    my_get = function(name, env=parent.frame(), inherits=TRUE) {

        if (identical(env, emptyenv()))
            stop("Can't find ", name, call. = FALSE)
        else if (exists(name, envir=env, inherits=FALSE))
            env[[name]]
        else if (inherits)
            my_get(name, env=parent.env(env))
        else
            stop("Can't find ", name, call. = FALSE)
    }

    my_get('x')
    #> Error: Can't find x

    x = 1
    my_get('x')
    #> [1] 1

    my_get('mean')
    #> function (x, ...)
    #> UseMethod("mean")
    #> <bytecode: 0x7fcf148750e8>
    #> <environment: namespace:base>
    ```


Function environments
---------------------

1.  List the four environments associated with a function. What does each one
    do? Why is the distinction between enclosing and binding environments
    particularly important?

    

Binding names to values
-----------------------

1.  What does this function do? How does it differ from <<- and why might you
    prefer it?
    ```r
    rebind <- function(name, value, env = parent.frame()) {
      if (identical(env, emptyenv())) {
        stop("Can't find ", name, call. = FALSE)
      } else if (exists(name, envir = env, inherits = FALSE)) {
        assign(name, value, envir = env)
      } else {
        rebind(name, value, parent.env(env))
      }
    }
    rebind("a", 10)
    #> Error: Can't find a
    a <- 5
    rebind("a", 10)
    a
    #> [1] 10
    ```

    `rebind` will emit an error if it cannot find `name` in the search path, while `<<-` will create a name in the global environment and bind the supplied value to it. This is 

