**Concept**: memoize the function and prevent it from being recreated on each rerender, thereby enhancing performance. The dep array may include a specific parameter necessary for the function to operate (e.g. id), so that the function gets recreated only when this parameter changes.
Particularly useful when passing callbacks to children or when using callbacks as dep in other hooks.

>Example 1. useCallback() ensures the function is only recreated when its prop in the dep array changes -> enhanced performance through reduction of unnecessary calculations.

```ts
const ArticleEditor = ({ id }: { id: string }) => {
  const submitChange = useCallback(
    async (summary: string) => {
      try {
        await fetch(`/api/articles/${id}`, {
          method: "POST",
          body: JSON.stringify({ id, summary }),
          headers: {
            "Content-Type": "application/json",
          },
        });
      } catch (error) {
        // handling errors
      }
    },
    [id]
  );
  return (
    <div>
      <ArticleForm onSubmit={submitChange} />
    </div>
  );
};
```

>Example 2

```ts
const ArticleForm = ({ onSubmit }: { onSubmit: (summary: string) => void }) => {
  const [summary, setSummary] = useState<string>("");
  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();
    onSubmit(summary);
  };
  const handleSummaryChange = useCallback(
    (event: ChangeEvent<HTMLTextAreaElement>) => {
      setSummary(event.target.value);
    },
    []
  );
  return (
    <form onSubmit={handleSubmit}>
      <h2>Edit Article</h2>
      <textarea value={summary} onChange={handleSummaryChange} />
      <button type="submit">Save</button>
    </form>
  );
};
```

[[React]]