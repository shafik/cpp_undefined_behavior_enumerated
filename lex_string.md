- The effect of attempting to modify a string literal is undefined
    - Example:
    
    ```cpp
    const char *p1 = "hello world\n"; 
    char *p2 = const_cast<char*>(p1) ;   // const_cast is already suspicious
    
    p2[0] = 'm' ;
    ```
   
    - Rationale *\[lex.string\]p8
    > Ordinary string literals and UTF-8 string literals are also referred to as narrow string literals. A narrow
string literal **has type “array of n const char”**, where n is the size of the string as defined below, and has
static storage duration (6.6.4).
