Vectors
-------

 1. What are the six types of atomic vector? How does a list differ from an atomic vector?
    
    Logical, integer, double, character, 
    
    A list can have elements of different types.

 2. What makes `is.vector()` and `is.numeric()` fundamentally different to `is.list()` and `is.character()`?
 
    `is.vector()` and `is.numeric()` match more than one data structure: `is.vector()` matches both atomic 
    vectors and lists, `is.numeric()` matches both integer and double.
    
 3. Test your knowledge of vector coercion rules by predicting the output of the following uses of c():
 
    ```r
    c(1, FALSE)
    c("a", 1)
    c(list(1), "a")
    c(TRUE, 1L)
    ```
    
    ```r
    c(1, FALSE)      # c(1, 0)
    c("a", 1)        # c("a", "1")
    c(list(1), "a")  # list(1, "a")
    c(TRUE, 1L)      # c(1, 1)
    ```
    
 4. Why do you need to use unlist() to convert a list to an atomic vector? Why doesn’t as.vector() work?
 
    Need to unwrap recursive structure.

 5. Why is 1 == "1" true? Why is -1 < FALSE true? Why is "one" < 2 false?
 
    1 -> "1", FALSE -> 0, 2 -> "2" and "2" appears before "o" in ASCII.

 6. Why is the default missing value, NA, a logical vector? What’s special about logical vectors? (Hint: think about c(FALSE, NA_character_).)
 
    That way it can be coerced up the type heirarchy Logical -> Integer -> Double -> Character, rather than
    forcing the other data to be coerced up.


Attributes
----------

 1. An early draft used this code to illustrate structure():
    ```r
    structure(1:5, comment = "my attribute")
    #> [1] 1 2 3 4 5
    ```
    But when you print that object you don’t see the comment attribute. Why? Is the attribute missing, or is there        something else special about it?
    
    `comment` (as well as `class`, `dim`, `dimnames`, `names`, `row.names` and `tsp`) are reserved and handled
    differently to other attributes.
    
 2. What happens to a factor when you modify its levels?
    ```r
    f1 <- factor(letters)
    levels(f1) <- rev(levels(f1))  
    ```
    
    The underlying integer values in any existing data isn't modified, but the association to names is.
    
 3. What does this code do? How do f2 and f3 differ from f1?
    ```r
    f2 <- rev(factor(letters))
    
    f3 <- factor(letters, levels = rev(letters))
    ```
    
    The integer values in `f2` are reversed from `f1`, as is the mapping from integers to names. 
    `f3` has the same integer data as `f2`---`c(26, 25, ..., 1)`---but the same mapping as `f1`.
 

Matrices and arrays
-------------------

 1. What does dim() return when applied to a vector?

    `NULL`

 2. If `is.matrix(x)` is `TRUE`, what will `is.array(x)` return?

    `TRUE`

    3. How would you describe the following three objects? What makes them different to 1:5?
    ```r
    x1 <- array(1:5, c(1, 1, 5))
    x2 <- array(1:5, c(1, 5, 1))
    x3 <- array(1:5, c(5, 1, 1))
    ```r