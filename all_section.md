# [lex]
## [lex.phases]
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

## [lex.pptoken]
- A preprocessing token is the minimal lexical element of the language in translation phases 3 through 6. … If a **’** or a **"** character matches the last category, the behavior is undefined. *\[lex.pptoken\]/p2*
    - Example [Why can't we use the preprocessor to create custom-delimeted strings?](https://stackoverflow.com/q/16798853/1708801)

```cpp
#define STR_START "
#define STR_END "

int puts(const char *);

int main() {
    puts(STR_START hello world STR_END);
}
```

- Discussion 
    - [Should it be ill-formed](https://groups.google.com/a/isocpp.org/forum/#!msg/std-discussion/lk1qAvCiviY/Zer-8iL88rUJ)
- Rationale
    - Preprocessing token are generated during phase 3 so string and character literals as well and header names are identified at this point. both ‘ and “ would only be valid as part of one of these tokens, any left over after this point perhaps as part of a macro would not be valid since they could not tokenized as needed anymore. Macros are expanded as part of phase 4.
        
## [lex.string]
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

# [basic]
## [basic.def.odr]
- There can be more than one definition of a class type (Clause 12), enumeration type (10.2), inline function with external linkage (10.1.6), inline variable with external linkage (10.1.6), class template (Clause 17), non-static function template (17.6.6), concept (17.6.8), static data member of a class template (17.6.1.3), member function of a class template (17.6.1.1), or template specialization for which some template parameters are not specified (17.8, 17.6.5) in a program provided that each definition appears in a different translation unit, and provided the definitions satisfy the following requirements. Given such an entity named D defined in more than one translation unit, then ... If the definitions of D do not satisfy these requirements, then the behavior is undefined. *\[basic.def.odr\]/p12*
    - [Examples from: DCL60-CPP. Obey the one-definition rule](https://wiki.sei.cmu.edu/confluence/display/cplusplus/DCL60-CPP.+Obey+the+one-definition+rule)
    
    ```cpp
    // a.cpp
    struct S {
      int a;
    };

    // b.cpp
    // S in b.cpp does not consist of the same sequence of token as S in a.cpp
    class S {
    public:
      int a;
    };
    ```


    ```cpp
    const int n = 42;

    int g(const int &lhs, const int &rhs);

    inline int f(int k) {
      return g(k, n);   // f() has external linkage 
             // n has internal linkage but is our-used by g() 
                                // n will not be identical in all transition units
    }
    ```



    - Rationale: 
        - [Devirtualization in C++, part 7 (Enforcing One Definition Rule) ](http://hubicka.blogspot.com/2014/09/devirtualization-in-c-part-6-enforcing.html)

## [basic.life]
- A program may end the lifetime of any object by reusing the storage which the object occupies or by explicitly calling the
destructor for an object of a class type with a non-trivial destructor. For an object of a class type with a non-trivial destructor,
the program is not required to call the destructor explicitly before the storage which the object occupies is reused or released;
however, **if there is no explicit call to the destructor or if a delete-expression ([expr.delete]) is not used to release the storage,
the destructor shall not be implicitly called and any program that depends on the side effects produced by the destructor has undefined behavior.**
    - [Pull request 2342 indicates this may not be undefined behavior at all](https://github.com/cplusplus/draft/pull/2342) and seeks the following edit<BR>
        <s>implicitly called and any program that depends on the side effects produced by the destructor has undefined behavior</s><u>implicitly called</u>.
        
## [basic.indet]
- If an indeterminate value is produced by an evaluation, the behavior is undefined except in the following
cases ... \[basic.indet\]p2
    - Example
    ```cpp
    int f(bool b) {
      unsigned char c;
      unsigned char d = c; // OK, d has an indeterminate value
      int e = d; // undefined behavior
      return b ? d : 0; // undefined behavior if b is true
    }
    ```
    - Rationale 
        - [WG14 Defect report 260](http://www.open-std.org/jtc1/sc22/wg14/www/docs/dr_260.htm)
        - [WG14 Defect report 451](http://www.open-std.org/Jtc1/sc22/WG14/www/docs/dr_451.htm)
        - Tl;DR; We have two case one in which using an indeterminate value is undefined behavior and this is because many type can have trap representations and using these value are undefined behavior. In the case of narrow character types the underlying values and type representation are one to one and therefore we don’t have a trap representation but they do retain their indeterminateness.

## [basic.start]

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
    
# [expr]

## [expr.pre]
- Signed integer overflow/underflow is undefined behavior
  - [\[expr.pre\]](http://eel.is/c++draft/expr#pre-4)
  - Examples
    ```cpp
    int x1=std::numeric_limits<int>::max()+1;
    int x2=std::numeric_limits<int>::min()-1;
    int x3=std::numeric_limits<int>::min() / -1;
    ```
    - [Examples live 1](https://godbolt.org/z/4M9uR7) and [examples live 2](https://godbolt.org/z/3ToiDL)

## [conv.double]

- Converting floating point value to type that cannot represent the value is undefined behavior even for float
  - [\[conv.double\]p1](https://timsong-cpp.github.io/cppwp/n4659/conv.double#1)
  - Examples
    ```cpp
    double d2=DBL_MAX;
    float f=d2;
    ```
    - [examples live](https://godbolt.org/z/p1y5JK)

## [conv.fpint]

- Converting floating point value to an integral that cannot represent the value is undefined behavior
  - [\[conv.fpint\]p1]( https://timsong-cpp.github.io/cppwp/n4659/conv.fpint#1)
  - Examples
    ```cpp
    double d=(double)INT_MAX+1;
    int x=d;
    ```
      - [Examples live](https://godbolt.org/z/8p0t_C)

## [expr.call]
- Calling a function through an expression whose function type is different from the function type of the called
function’s definition results in undefined behavior
  - [\[expr.call\]p6]( http://eel.is/c++draft/expr.call#6)
  - Examples
  ```cpp
  int f_c(int);

  using c1 = int(*)(int);
  using c2 = int(*)(int,int);

  int f(c2 func) {
   return func(1,2);
  }

  int main() {
    f(reinterpret_cast<c2>(f_c));
  }
  ```
    - [Examples  live](https://gcc.godbolt.org/z/2gUEol)

## [expr.static.cast]
- If the object of type “cv1 B” is actually a base class subobject of an object of type D, the result refers to the
enclosing object of type D. Otherwise, the behavior is undefined.
  - [\[expr.static.cast\]](http://eel.is/c++draft/expr.static.cast#2)
  - Examples:
  ```cpp
  struct B {};
  struct D1:B {};
  struct D2:B {};

  void f() {
   D1 d;
   B &b = d;
   static_cast<D2&>(b);
  }
  ```
    - [Examples live](https://godbolt.org/z/z7RTFJ)


