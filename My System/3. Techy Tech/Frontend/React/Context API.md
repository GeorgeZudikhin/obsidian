
> [!NOTE] Idea
> Context is a valuable tool to create a global state that can be accessed by components throughout the application. Use **separate context providers**: for security, logging etc. to organize logic and share functionality in an encapsulated and efficient way.

>Example:

```ts
import InteractionContext from "xui/interaction-context";
import SecurityContext from "xui/security-context";
import LoggingContext from "xui/logging-context";
const Application = ({ children }) => {
  const context = {}; // ... define values for context
  //... define securityContext
  //... define loggingContext
  return (
    <InteractionContext.Provider value={context}>
      <SecurityContext.Provider value={securityContext}>
        <LoggingContext.Provider value={loggingContext}>
          {children}
        </LoggingContext.Provider>
      </SecurityContext.Provider>
    </InteractionContext.Provider>
  );
};
```

[[React]]