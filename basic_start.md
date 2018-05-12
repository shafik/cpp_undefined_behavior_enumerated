
- A hosted implementation shall contain a global function called main in one of the specified forms *\[basic.start.main\]*
    - 6.8.3.1
    - Example:

    ```cpp
    void main() {}
    ```
    - Rationale:
    
    - Tools
        - [Compiler static analysis, generates warnings or errors for void main](https://wandbox.org/permlink/9YlvHEA88lS0CbvV)

- The function main shall not be used within a program   *\[basic.start.main\]*
    - 6.8.3.1
    - Examples:
        - [Is it illegal to take address of main() function?](https://stackoverflow.com/q/15525613/1708801)
        - [Why does gcc warn about decltype(main()) but not clang?](https://stackoverflow.com/q/25297257/1708801)
        
        ```cpp
        printf( “%p\n”, &main ) ;
        decltype(main()) x = 0;
        ```
        
        ```cpp
        int main() {
          std::cout << reinterpret_cast<void*>(&main) ;
        }
        ```
        
    - Tools
        - [Compiler static analysis via -pedantic flag gcc/clang for taking address of main](https://wandbox.org/permlink/atm7dJ1LXaVBkCsU)
        - [Compiler static analysis via -pedantic flag gcc/clang decltype main](https://wandbox.org/permlink/WvPFbU4hteDOX4w2)

    - Rationale: 
        - From ARM section 3.4 Start and Termination
            > This is to ensure full freedom of the implementation of the interface between a C++ program and its environment.
            > One could imagine an implementation where main() was not implemented as a function.
        - [\[ub\] What does "The function main shall not be used within a program" mean?](http://www.open-std.org/pipermail/ub/2014-January/000474.html)
    
