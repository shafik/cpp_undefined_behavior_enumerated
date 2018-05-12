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
        
