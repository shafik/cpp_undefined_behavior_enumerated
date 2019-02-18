- A program may end the lifetime of any object by reusing the storage which the object occupies or by explicitly calling the
destructor for an object of a class type with a non-trivial destructor. For an object of a class type with a non-trivial destructor,
the program is not required to call the destructor explicitly before the storage which the object occupies is reused or released;
however, **if there is no explicit call to the destructor or if a delete-expression ([expr.delete]) is not used to release the storage,
the destructor shall not be implicitly called and any program that depends on the side effects produced by the destructor has undefined behavior.**
    - [Pull request 2342 indicates this may not be undefined behavior at all](https://github.com/cplusplus/draft/pull/2342) and seeks the following edit<BR>
        <s>implicitly called and any program that depends on the side effects produced by the destructor has undefined behavior</s><u>implicitly called</u>.
