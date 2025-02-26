Often, a complex component (e.g. a page) needs to accept multiple props and pass them to several children. This results in an unwieldy amount of props, potential confusion, poor testing experience, and low maintainability, especially if new props were to be added. 
In this case, instead of passing individual props that will just be prop drilled separately to child components, the parent can accept entire components as React Nodes. This makes the parent highly configurable with customized components.

> Instead of this

```ts
function Page({
  headerTitle,
  headerSubtitle,
  sidebarLinks,
  mainContent,
  isLoading,
  onHeaderClick,
  onSidebarLinkClick,
}: PageProps) {
  return (
    <div>
      <Header
        title={headerTitle}
        subtitle={headerSubtitle}
        onClick={onHeaderClick}
      />
      <Sidebar links={sidebarLinks} onLinkClick={onSidebarLinkClick} 
       />
      <Main isLoading={isLoading} content={mainContent} />
    </div>
  );
}
```

> Try this

```ts
type PageProps = {
  header: React.ReactNode;
  sidebar: React.ReactNode;
  main: React.ReactNode;
};

function Page({ header, sidebar, main }: PageProps) {
  return (
    <div>
      {header}
      {sidebar}
      {main}
    </div>
  );
}

function MyPage () {
  return (
    <Page
      header={
        <Header
          title="My application"
          subtitle="Product page"
          onClick={() => console.log("toggle header")}
        />
      }
      sidebar={
        <Sidebar
          links={["Home", "About", "Contact"]}
          onLinkClick={() => console.log("toggle sidebar")}
        />
      }
      main={<Main isLoading={false}
            content={<div>The main</div>} />}
    />
  );
};
```

[[React Antipatterns]]