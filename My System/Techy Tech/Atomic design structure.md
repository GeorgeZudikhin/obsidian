Application is broken down into a hierarchy of components. The lower a component is placed in the hierarchy, the simpler and more trivial nature it possesses. A component from each subsequent level in the hierarchy is made up of its lower-level predecessors.
1. **Atoms** - smallest UI building blocks, represent individual self-contained elements, e.g. buttons, inputs, icons, labels
2. **Molecules** - encapsulate a group of atoms to form a functional unit, e.g. a form field or a navigation bar
3. **Organisms** - combine molecules & atoms to create more significant UI sections, e.g. header, sidebar, card component
4. **Templates** - a layout structure for arranging organisms & molecules that defines the overall skeleton of a page or a specific UI section
5. **Pages** - complete UI screens composed of the preceding components in the hierarchy

![[Atomic design structure.png]]

Pros:
- maintainability
- reusability
- consistency

Cons: 
- complexity, terrible fucking overengineering

[[React App Project Structure]]