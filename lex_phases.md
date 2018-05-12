- if a splice results in a character sequence that matches the syntax of a universal-character-name, the behavior is undefined. *\[lex.phases\]/p2*
    - Example from [Stacoverflow question](https://stackoverflow.com/q/43824729/1708801):

    ```cpp
    const char* p = "\\
    u0041";
    ```

    - Rationale
        - [DR 787](http://www.open-std.org/jtc1/sc22/wg21/docs/cwg_defects.html#787)
        - [N3881 “Fixing the specification of universal-character-names”](http://open-std.org/jtc1/sc22/wg21/docs/papers/2014/n3881.pdf)

- If a character sequence that matches the syntax of a universal-character-name is produced by token concatenation (19.3.3), the behavior is undefined. *\[lex.phases\]/p4*
    - example:  

    ```cpp
    #define GUARD_NAME ï ## _GUARD // UB per current spec
    #define COLUMN "ï" ## _column // UB per current spec
    ```
    
    - Rationale
        - [DR 787](http://www.open-std.org/jtc1/sc22/wg21/docs/cwg_defects.html#787)
        - [N3881 “Fixing the specification of universal-character-names”](http://open-std.org/jtc1/sc22/wg21/docs/papers/2014/n3881.pdf)

