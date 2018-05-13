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
        
