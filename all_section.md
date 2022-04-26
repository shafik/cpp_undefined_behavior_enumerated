# [lex]
## [lex.phases]
- if a splice results in a character sequence that matches the syntax of a universal-character-name, the behavior is undefined. 
  - [\[lex.phases\]p1.2](http://eel.is/c++draft/lex.phases#1.2)
  - Example from [Stack Overflow question](https://stackoverflow.com/q/43824729/1708801):
  ```cpp
  const char* p = "\\
  u0041";
  ```
  - [Examples live](https://godbolt.org/z/xZLefc)
  - Rationale
    - [DR 787](http://www.open-std.org/jtc1/sc22/wg21/docs/cwg_defects.html#787)
    - [N3881 “Fixing the specification of universal-character-names”](http://open-std.org/jtc1/sc22/wg21/docs/papers/2014/n3881.pdf)

- If a character sequence that matches the syntax of a universal-character-name is produced by token concatenation (19.3.3), the behavior is undefined. *\[lex.phases\]/p4*
  - [\[lex.phases\]p1.4](http://eel.is/c++draft/lex.phases#1.4)
  - Examples:  
  ```cpp
  #define GUARD_NAME ï ## _GUARD // UB per current spec
  #define COLUMN "ï" ## _column // UB per current spec
  ```
  - [Examples live](https://godbolt.org/z/BO8PGi)
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
  - [Examples live](https://godbolt.org/z/mqiAJw)
  - Discussion 
    - [Should it be ill-formed](https://groups.google.com/a/isocpp.org/forum/#!msg/std-discussion/lk1qAvCiviY/Zer-8iL88rUJ)
  - Rationale
    - Preprocessing token are generated during phase 3 so string and character literals as well and header names are identified at this point. both ‘ and “ would only be valid as part of one of these tokens, any left over after this point perhaps as part of a macro would not be valid since they could not tokenized as needed anymore. Macros are expanded as part of phase 4.
        
## [lex.string]
- The effect of attempting to modify a string literal is undefined
  - Examples:
  ```cpp
  const char *p1 = "hello world\n"; 
  char *p2 = const_cast<char*>(p1) ;   // const_cast is already suspicious
    
  p2[0] = 'm' ;
  ``` 
  - [Examples live](https://godbolt.org/z/wrNAzz)
  - Rationale *\[lex.string\]p8
    > Ordinary string literals and UTF-8 string literals are also referred to as narrow string literals. A narrow
string literal **has type “array of n const char”**, where n is the size of the string as defined below, and has
static storage duration (6.6.4).

# [basic]
## [basic.def.odr]
- There can be more than one definition of a class type (Clause 12), enumeration type (10.2), inline function with external linkage (10.1.6), inline variable with external linkage (10.1.6), class template (Clause 17), non-static function template (17.6.6), concept (17.6.8), static data member of a class template (17.6.1.3), member function of a class template (17.6.1.1), or template specialization for which some template parameters are not specified (17.8, 17.6.5) in a program provided that each definition appears in a different translation unit, and provided the definitions satisfy the following requirements. Given such an entity named D defined in more than one translation unit, then ... If the definitions of D do not satisfy these requirements, then the behavior is undefined. 
  - [\[basic.def.odr\]p12](http://eel.is/c++draft/basic.def.odr#12)
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
  - [\[basic.life\]p5](http://eel.is/c++draft/basic.life#5)
  - [Pull request 2342 indicates this may not be undefined behavior at all](https://github.com/cplusplus/draft/pull/2342) and seeks the following edit<BR>
        <s>implicitly called and any program that depends on the side effects produced by the destructor has undefined behavior</s><u>implicitly called</u>.
        
## [basic.indet]
- If an indeterminate value is produced by an evaluation, the behavior is undefined except in the following
cases 
  - [\[basic.indet\]p2](http://eel.is/c++draft/basic.indet#2)
  - Examples
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

- An implementation shall not predefine the main function. This function shall not be overloaded. Its type shall have C++ language linkage and it shall have a declared return type of type int, but otherwise its type is implementation-defined.
    - [\[basic.start.main\]p2](http://eel.is/c++draft/basic.start#main-2)
    - Examples:
    ```cpp
    void main() {}
    ```
    - Tools
      - [Compiler static analysis, generates warnings or errors for void main](https://wandbox.org/permlink/9YlvHEA88lS0CbvV)

- The function main shall not be used within a program  
    - [\[basic.start.main\]p3](http://eel.is/c++draft/basic.start#main-3)
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
  
- Setting an enum to a value outside the range of enumerators is undefined behavior
  - [\[expr.static.cast\]p10](http://eel.is/c++draft/expr.static.cast#10) and [\[dcl.enum\]p8](http://eel.is/c++draft/dcl.enum#8)
  - Examples
  ```cpp
  enum A {e1=1, e2};

  void f() {
    enum A a=static_cast<A>(4);   
  } 
  ```
  - [Examples live](https://wandbox.org/permlink/YWXO1IQt3DLSSHmb)
  
- Down-casting to the wrong derived type is undefined behavior
  - [\[expr.static.cast\]p11](http://eel.is/c++draft/expr.static.cast#11)
  - Examples
  ```cpp
  struct B {};
  struct D1:B {};
  struct D2:B {};

  void f() {
    B* bp = new D1;
    static_cast<D2*>(bp);
  }
  ```
  - [Examples lives](https://godbolt.org/z/gfT5Bw)

## [expr.delete]
- Using array delete on the result of a single object new expression and vice versa is undefined behavior
  - [\[expr.delete\]p2])http://eel.is/c++draft/expr.delete#2)
  - Examples
  ```cpp
  int *x = new int;
  delete [] x;
  ```
  - [Examples live](https://godbolt.org/z/KI8XSc)
  
- If the dynamic type differs from the static type of the object being deleted that is undefined behavior
  - [\[expr.delete\]p3](http://eel.is/c++draft/expr.delete#3)
  - Examples
  ```cpp
  int *p = new int;
  float *f = reinterpret_cast<float*>(p);
  delete f;
  ```
  - [Examples lives](https://godbolt.org/z/Pj_Ljb)
  
- Deleting and incomplete type and the class turns out to have a non-trivial destructor is undefined behavior
  - [\[expr.delete\]p5](http://eel.is/c++draft/expr.delete#5)
  - Examples
  ```cpp
  struct A;

  void f(A *p) {
    delete p;
  }

  struct A {~A(){}};
  ```
  - [Examples live](https://godbolt.org/z/Jc6lKv)
  
## [expr.mptr.oper]
- If the dynamic type of E1 does not contain the member to which E2 refers, the behavior is undefined
  - [\[expr.mptr.oper\]p4](http://eel.is/c++draft/expr.mptr.oper#4)
  - Examples:
  ```cpp
  struct B{};
  struct D:B{int x;};

  void f(){
   B *b= new B;
   D *d=static_cast<D*>(b);
   int D::* p=&D::x;
   (*d).*p=1;
  }
  ```
  - [Examples live](https://godbolt.org/z/wgNkKz)
  
- If the second operand is the null member pointer value (7.3.12), the behavior is undefined.
  - [\[expr.mptr.oper\]p6](http://eel.is/c++draft/expr.mptr.oper#6)
  - Examples:
  ```cpp
  struct S {
   int i;
  };
  
  void f()
  {
   S cs;
   int S::* pm = nullptr;
   cs.*pm = 88;
  }
  ```
  - [Examples live](https://godbolt.org/)

## [expr.mul]
- Divison by zero is undefined behavior
  - [\[expr.mul\]p4](http://eel.is/c++draft/expr.mul#4)
  - Examples:
  ```cpp
  int x = 1/0;
  double d = 1.0/0.0;
  ```
  - [Examples live](https://godbolt.org/z/d42Fsi)
  
## [expr.add]
- Incrementing pointer beyond one past the end of an array is undefined behavior
  - [\[expr.add\]p4](http://eel.is/c++draft/expr.add#4) and [footnote](http://eel.is/c++draft/expr.add#footnote-80)
  - Examples:
  ```cpp
  static const int arrs[10]{};

  void f() {
   const int* y = arrs + 11;
  }
  ```
  - [Examples live](https://godbolt.org/z/Oo9lWi)
  
  
- Subtracting pointers that are not part of the same array is undefined behavior
  - [\[expr.add\]p5.3](http://eel.is/c++draft/expr.add#5.3)
  - Examples:
   ```cpp
   void f() {
    int x;
    int y;
    int *p1=&x;
    int *p2=&y;
    std::ptrdiff_t off = p1-p2;
   }
   ```
   - [Examples live](https://godbolt.org/z/BxwQjE)
   
 ## [expr.shift]
- Shifting by a negative amount is undefined behavior
  - [\[expr.shift\]p1](http://eel.is/c++draft/expr.shift#1)
  - Examples
  ```cpp
  int y = 1 << -1;
  ```
  - [Examples live](https://godbolt.org/z/op_tEL)
   
- Shifting by equal or greater than the bit-width of a type is undefined behavior
  - [\[expr.shift\]p1](http://eel.is/c++draft/expr.shift#1)
  - Examples:
  ```cpp
  int y1 = 1 << 32;
  int y2 = 1 >> 32;
  ```
  - [Examples live](https://godbolt.org/z/fx156-)
   
- Shifting a negative signed type is undefined behavior (before C++20)
  - [\[expr.shift\]p2](https://timsong-cpp.github.io/cppwp/n4659/expr.shift#2)
  - Examples:
  ```cpp
  int y4 = -1 << 12;
  ```
  - [Examples live](https://godbolt.org/z/v3B1ij)
   
   
## [expr.ass]
- Overlap in an assignment expression must be exact and the objects must have the same type
  - [\[expr.ass\]p8](http://eel.is/c++draft/expr.ass#8)
  - Examples:
  ```cpp
  int x=1;
  char *c=reinterpret_cast<char*>(&x);
  x = *c;
  ```
  - [Examples live](https://godbolt.org/z/LGBL7k)
  
# [stmt.stmt]
## [stmt.return]
- Flowing off the end of a value returning function is undefined behavior
  - [\[stmt.return\]p2](http://eel.is/c++draft/stmt.return#2.sentence-8)
  - Examples:
  ```cpp
  int f(int x) {
  if(x)
    return 1;
  }
 
  void b(){
   int x=f(0);
  }
  ```
  - [Examples live](https://godbolt.org/z/62CGnB)
  - [Also see]( https://twitter.com/shafikyaghmour/status/975224687444688896)
  
## [stmt.dcl]
- Recursively entering declaration of a block scope static variable during initialization is undefined behavior
  - [\[stmt.dcl\]p4](http://eel.is/c++draft/stmt.dcl#4)
  - Examples:
  ```cpp
  int foo(int i) {
  static int s = foo(2*i);
  return i+1;
  }
  ```
  - [Examples live](https://godbolt.org/z/mT9a-_)
  
# [dcl.dcl]
## [dcl.type.cv]
- Attempting to modify a const object is undefined behavior
  - [\[dcl.type.cv\]p4](http://eel.is/c++draft/dcl.type.cv#4)
  - Examples:
  ```cpp
  int bar() {
  const int x=1;

  int *p = const_cast<int*>(&x);
  *p = 2;

  return *p;
  }
  ```
  - [Examples live](https://godbolt.org/z/MArrbe)
  
- Accessing a volatile value through a non-volatile is undefined behavior
  - [\[dcl.type.cv\]p5](http://eel.is/c++draft/dcl.spec#dcl.type.cv-5)
  - Examples:
  ```cpp
  void f() {
  volatile int x=0;
  int &y=const_cast<int&>(x);
  std::cout << y;
  }
  ```
  -[Examples live](https://godbolt.org/z/4xKsxy)
  
  
## [dcl.attr.contract.syn]
- In contracts side effects in a predicate to an object whose lifetime did not begin and end within the evaluation of the predicate are undefined behavior
  - [\[dcl.attr.contract.syn\]p6](http://eel.is/c++draft/dcl.attr.contract#syn-6)
  - Examples:
  ```cpp
  int min = -42;
  constexpr int g(int x) {
    /* ... */
    [[assert: ++min > 0]]; // undefined behavior
    /* ... */
    return 0;
  }
  ```
  - [Examples live](http://fragata.arcos.inf.uc3m.es/#g:!((g:!((g:!((h:codeEditor,i:(j:1,lang:c%2B%2B,source:'int+min+%3D+-42%3B%0Aconstexpr+int+g(int+x)%7B%0A++/*+...+*/%0A++%5B%5Bassert:+%2B%2Bmin+%3E+0%5D%5D%3B+//+undefined+behavior%0A++/*+...+*/%0A++return+0%3B%0A%7D'),l:'5',n:'0',o:'C%2B%2B+source+%231',t:'0')),k:50,l:'4',n:'0',o:'',s:0,t:'0'),(g:!((g:!((h:compiler,i:(compiler:clang%2B%2B-master,filters:(b:'0',binary:'1',commentOnly:'0',demangle:'0',directives:'0',execute:'1',intel:'0',trim:'0'),lang:c%2B%2B,libs:!(),options:'',source:1),l:'5',n:'0',o:'Clang+6.0.0+x86_64+%5Bclang-contracts%5D+(master)+(Editor+%231,+Compiler+%231)+C%2B%2B',t:'0')),k:50,l:'4',m:50,n:'0',o:'',s:0,t:'0'),(g:!((h:output,i:(compiler:1,editor:1),l:'5',n:'0',o:'%231+with+Clang+6.0.0+x86_64+%5Bclang-contracts%5D+(master)',t:'0')),header:(),l:'4',m:50,n:'0',o:'',s:0,t:'0')),k:50,l:'3',n:'0',o:'',t:'0')),l:'2',n:'0',o:'',t:'0')),version:4)
  
## [dcl.attr.contract.syn]
- if a postcondition odr-uses a non-reference parameter in its predicate and the function body makes direct or indirect modifications of the value of that parameter, the behavior is undefined.
  - [\[dcl.attr.contract.cond\]p7](http://eel.is/c++draft/dcl.attr.contract#cond-7)
  - Examples:
  ```cpp
  int f(int x)
  [[ensures r: r == x]]
  {
    return ++x; // UB
  }
  ```
  - [Examples live](http://fragata.arcos.inf.uc3m.es/#g:!((g:!((g:!((h:codeEditor,i:(j:1,lang:c%2B%2B,source:'int+f(int+x)%0A++%5B%5Bensures+r:+r+%3D%3D+x%5D%5D%0A%7B%0A++return+%2B%2Bx%3B+//+UB%0A%7D%0A'),l:'5',n:'0',o:'C%2B%2B+source+%231',t:'0')),k:50,l:'4',n:'0',o:'',s:0,t:'0'),(g:!((g:!((h:compiler,i:(compiler:clang%2B%2B-master,filters:(b:'0',binary:'1',commentOnly:'0',demangle:'0',directives:'0',execute:'1',intel:'0',trim:'0'),lang:c%2B%2B,libs:!(),options:'',source:1),l:'5',n:'0',o:'Clang+6.0.0+x86_64+%5Bclang-contracts%5D+(master)+(Editor+%231,+Compiler+%231)+C%2B%2B',t:'0')),k:50,l:'4',m:50,n:'0',o:'',s:0,t:'0'),(g:!((h:output,i:(compiler:1,editor:1),l:'5',n:'0',o:'%231+with+Clang+6.0.0+x86_64+%5Bclang-contracts%5D+(master)',t:'0')),header:(),l:'4',m:50,n:'0',o:'',s:0,t:'0')),k:50,l:'3',n:'0',o:'',t:'0')),l:'2',n:'0',o:'',t:'0')),version:4)
  
## [dcl.attr.contract.check]
- Violating a non-checked contract is undefined behavior outside of a constant expression context
  - [\[dcl.attr.contract.check\]p4](http://eel.is/c++draft/dcl.attr.contract#check-4)
  - Rationale see [p1321r0](http://open-std.org/JTC1/SC22/WG21/docs/papers/2018/p1321r0.html) and [p1490r0](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1490r0.html)
  - Examples:
  ```cpp
  void f(int x) [[expects audit: x>=1 && x<=2]];
 
  void b() {
    f(100);
  }
  ```
  - [Examples live](http://fragata.arcos.inf.uc3m.es/#g:!((g:!((g:!((h:codeEditor,i:(j:1,lang:c%2B%2B,source:'void+f(int+x)+%5B%5Bexpects+audit:+x%3E%3D1+%26%26+x%3C%3D2%5D%5D%3B%0A%0Avoid+b()+%7B%0A++f(100)%3B%0A%7D'),l:'5',n:'0',o:'C%2B%2B+source+%231',t:'0')),k:50,l:'4',n:'0',o:'',s:0,t:'0'),(g:!((g:!((h:compiler,i:(compiler:clang%2B%2B-master,filters:(b:'0',binary:'1',commentOnly:'0',demangle:'0',directives:'0',execute:'1',intel:'0',trim:'0'),lang:c%2B%2B,libs:!(),options:'',source:1),l:'5',n:'0',o:'Clang+6.0.0+x86_64+%5Bclang-contracts%5D+(master)+(Editor+%231,+Compiler+%231)+C%2B%2B',t:'0')),k:50,l:'4',m:50,n:'0',o:'',s:0,t:'0'),(g:!((h:output,i:(compiler:1,editor:1),l:'5',n:'0',o:'%231+with+Clang+6.0.0+x86_64+%5Bclang-contracts%5D+(master)',t:'0')),header:(),l:'4',m:50,n:'0',o:'',s:0,t:'0')),k:50,l:'3',n:'0',o:'',t:'0')),l:'2',n:'0',o:'',t:'0')),version:4)
  
## [dcl.attr.noreturn]
- A function declared noreturn eventually returns it is undefined behavior
  - [\[dcl.attr.noreturn\]p2](http://eel.is/c++draft/dcl.attr.noreturn#2)
  - Examples:
  ```cpp
  [[ noreturn ]] void q(int i) { // behavior is undefined if called with an argument <= 0
  if (i > 0)
    throw "positive";
  }
  ```
  - [Examples live](https://godbolt.org/z/_8GQBg)
  
# [class]
  
## [class.mfct.non-static]
- Calling a non-static member function of a class with an object that is not of that type is undefined behavior
  - [\[class.mfct.non-static\]p2](http://eel.is/c++draft/class.mem#class.mfct.non-static-2)
  - Examples:
  ```cpp
  struct X {
    int x=1;
    int f() { return x;}
  };

  struct A {int x=3;};

  int f(X*x) {
    return x->f();
  }

  int main() {
    A a;
    std::cout<< f(reinterpret_cast<X*>(&a));
  }
  ```
  - [Examples live](https://godbolt.org/z/0zBLzy)
  
## [class.dtor]
- Explicit destructor call for an object not of the type is undefined behavior
  - [\[class.dtor\]p14](http://eel.is/c++draft/class.dtor#14)
  - Examples:
  ```cpp
  struct X {};

  void f() {
    X *x=nullptr;
    x->~X();
  }
  ```
  - [Examples live](https://godbolt.org/z/Qola5k)
  
- Invoking the destructor for an object once its lifetime has ended is undefined behavior
  - [\[class.dtor\]p16](http://eel.is/c++draft/class.dtor#16)
  - Examples:
  ```cpp
  struct A{
     ~A(){}
  };

  int main() {
    A a;
    a.~A(); // Destructor will be invoked again at scope exit invoking UB
  }
  ```
  - [Examples live](https://godbolt.org/z/kHMPig)
  
## [class.union]
- Accessing a non-active union member is undefined behavior
  - [\[class.union\]p1](http://eel.is/c++draft/class.union#1)
  - Examples:
  ```cpp
  union Y { float f; int k; };
  void g() {
   Y y = { 1.0f }; // OK, y.f is active union member (10.3)
   int n = y.k;
  }
  ```
  - [Examples live](https://godbolt.org/z/LbbRnS)
  
## [class.abstract]
- Calling a virtual function from a constructor or destructor in an abstract class is undefined behavior
  - [\[class.abstract\]p6](http://eel.is/c++draft/class.derived#class.abstract-6)
  - Examples:
  ```cpp
  struct B {
   virtual void f()=0;
   B() { f();}
  };

  struct D : B{
      void f() override { }
  };
  ```
  - [Examples live](https://godbolt.org/z/mFTX2B)
  
## [class.base.init]
- Calling a member function before all baes are initialized is undefined behavior
  - [\[class.base.init\]p16](http://eel.is/c++draft/class.base.init#16)
  - Examples:
  ```cpp
  struct B {
    B(int);
  };

  struct D : public B {
    int f();
    D() : B(f()) {}
  };
  ```
  - [Examples live](https://godbolt.org/z/um0iNu)
  
## [class.cdtor]
- For an object with a non-trivial constructor, referring to any non-static member or base class of the object
before the constructor begins execution results in undefined behavior
  - [\[class.cdtor\]p1](http://eel.is/c++draft/class.cdtor#1)
  - Examples
  ```cpp
  struct W { int j; };
  struct X : public virtual W { };
  
  struct Y {
   int* p;
   X x; 
   Y() : p(&x.j) { // undefined, x is not yet constructed
    }
  };
  ```
  - [Examples live](https://godbolt.org/z/fFbAY9)
  
- To explicitly or implicitly convert a pointer (a glvalue) referring to an object of class X to a pointer (reference)
to a direct or indirect base class B of X, the construction of X and the construction of all of its direct or
indirect bases that directly or indirectly derive from B shall have started and the destruction of these classes
shall not have completed, otherwise the conversion results in undefined behavior
  - [\[class.cdtor\]p3](http://eel.is/c++draft/class.cdtor#3)
  - Examples:
  ```cpp
  struct A { };
  struct B : virtual A { };
  struct C : B { };
  struct D : virtual A { D(A*); };
  struct X { X(A*); };

  struct E : C, D, X {
    E() : D(this), // undefined: upcast from E* to A* might use path E* ! D* ! A*
                       // but D is not constructed

                      // “D((C*)this)” would be defined: E* ! C* is defined because E() has started,
                     // and C* ! A* is defined because C is fully constructed

      X(this) {} // defined: upon construction of X, C/B/D/A sublattice is fully constructed. 
  };
  ```
  - [Examples live](https://godbolt.org/z/Xu8yOi)
  
- To form a pointer to (or access the value of) a direct non-static member of an object obj, the construction of obj shall have started and its destruction shall not have completed, otherwise the computation of the pointer value (or accessing
the member value) results in undefined behavior. 
  - [\[class.cdtor\]p3](http://eel.is/c++draft/class.cdtor#3)
  - Examples:
  ```cpp
  struct A {
    int x;
  };

  void f() {
    A a;
    a.~A();
    int *p=&a.x; // Destruction completed so computing the pointer is undefined behavior
  }
  ```
  - [Examples lives](https://godbolt.org/z/O89aee)
  
- If the virtual function call uses an explicit class member access (7.6.1.4) and the object expression refers to the complete
object of x or one of that object’s base class subobjects but not x or one of its base class subobjects, the
behavior is undefined.
  - [\[class.cdtor\]p4](http://eel.is/c++draft/class.cdtor#4)
  - Examples:
  ```cpp
  struct V {
    virtual void f();
    virtual void g();
  };

  struct A : virtual V {
    virtual void f();
  };

  struct B : virtual V {
    virtual void g(); 
    B(V*, A*);
  };

  struct D : A, B {
    virtual void f();
    virtual void g();
    D() : B((A*)this, this) { }
  };

  B::B(V* v, A* a) {
    f(); // calls V::f, not A::f
    g(); // calls B::g, not D::g
    v->g(); // v is base of B, the call is well-defined, calls B::g
    a->f(); // undefined behavior, a’s type not a base of B. 
  }
  ```
  - [Examples live](https://godbolt.org/z/-U8W-2)
  
- If the operand of typeid refers to the object under construction or destruction and the static type of the operand is neither the constructor or destructor’s class nor one of its bases, the behavior is undefined
  - [\[class.cdtor\]p5](http://eel.is/c++draft/class.cdtor#5)
  - Examples:
  ```cpp
  struct V {
   virtual void f();
  };

  struct A : virtual V { };
  struct B : virtual V {
   B(V*, A*);
  };

  struct D : A, B {
   D() : B((A*)this, this) { }
  };

  B::B(V* v, A* a) {
    typeid(*this); // type_info for B.
    typeid(*v); // well-defined: *v has type V, a base of B yields type_info for B
    typeid(*a); // undefined behavior: type A not a base of B
    dynamic_cast<B*>(v); // well-defined: v of type V*, V base of B results in B*
    dynamic_cast<B*>(a); // undefined behavior, a has type A*, A not a base of B
  }
  ```
  - [Examples live](https://godbolt.org/z/XH7xat)
  
- If the operand of the dynamic_cast refers to the object under construction or destruction and the static type
of the operand is not a pointer to or object of the constructor or destructor’s own class or one of its bases,
the dynamic_cast results in undefined behavior
  - [\[class.cdtor\]p6](http://eel.is/c++draft/class.cdtor#6)
  - Examples:
  ```cpp
    struct V {
   virtual void f();
  };

  struct A : virtual V { };
  struct B : virtual V {
   B(V*, A*);
  };

  struct D : A, B {
   D() : B((A*)this, this) { }
  };

  B::B(V* v, A* a) {
    typeid(*this); // type_info for B.
    typeid(*v); // well-defined: *v has type V, a base of B yields type_info for B
    typeid(*a); // undefined behavior: type A not a base of B
    dynamic_cast<B*>(v); // well-defined: v of type V*, V base of B results in B*
    dynamic_cast<B*>(a); // undefined behavior, a has type A*, A not a base of B
  }
  ```
  - [Examples live](https://godbolt.org/z/XH7xat)
