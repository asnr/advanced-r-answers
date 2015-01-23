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
