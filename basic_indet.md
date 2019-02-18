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
