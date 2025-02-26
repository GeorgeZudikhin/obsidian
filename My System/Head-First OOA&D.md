# Well-Designed Apps Rock
### Procedure when delivering new functionality
1. Make sure the software works and completely fulfills the customer’s requirements. Some initial design thinking is okay at this point, but focus solely on the feature working well
2. Apply basic OO principles to add flexibility
3. Apply patterns and principles to achieve maintainable, reusable design

### Encapsulation
Encapsulation is about taking a part of code that is reused often and storing it separately. This way, it can be referenced from all places that would otherwise have to redefine it. If this encapsulated code needs to be extended, it can simply be extended in where it is stored, without have to change the code that references it. Encapsulation enforces clear boundaries that allow classes to hide their inner behavior.

One of the most important OO principles - *delegation*. Make sure the class is self-contained and performs the logic only on itself. If class B needs specific logic from class A, this logic can be implemented within class A, instead of class B accessing properties of class A and trying to mold something together. This way, in case any behavior of this logic needs to be changed, it can be changed only in class A. Class B will not be changed, since it has no idea of how the logic is implemented in class A and simply accesses the output. This leads to the concept of *loose coupling* - when classes have a specific job and they do only that job. Adding new functionality becomes easier, because only individual classes need to be changed.