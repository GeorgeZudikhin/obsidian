The application is organized into distinct features i.e. modules. Each feature gets its own folders: components, containers, pages, services, utils etc. 
**shared** folder contains common reusable components, containers etc
**API** folder configures logic for making API calls
**store** folder contains modules for state management
**router** contains routing configuration and related components
**App.tsx** serves as the entry point of the application

![[Feature-based structure.png]]

Pros:
- modularity
- separation of concerns
- easy team collaboration with no overlaps
- high maintainability

Cons:
- likely code duplications in case with similar features

[[React App Project Structure]]