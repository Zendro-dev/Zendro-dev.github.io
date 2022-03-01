[ &larr; back](setup_root.md)
# Customizing Zendro
Zendro offers multiple different ways to customize it to your specific needs.
## Graphql server
### Custom validations
It is possible to add custom asynchronous validation functions to validate your data. For more information see:  

[> custom validations](https://zendro-dev.github.io/setup_data_scheme.html#custom-validator-function-for-ajv)
### Patches
Custom patches allow the user to monkey patch functions and properties of the generated backend code. When running the code generator a skeleton patch file is automatically created for every data-model in `/patches`. Implementing functions there will automatically override default behaviour. An example patch to override `<function_name>` could look something like this:

```
data.prototype.<function_name> = function(...) {...}
```

## Single page application
### Custom Pages

Routes in the application automatically mirror the structure of the pages folder. The default static site contains a dynamic `[model]` route, with a home page ([`index.tsx`](./src/pages/[model]/index.tsx)) to display an interactive table of records, and one child route ([`item.tsx`](./src/pages/[model]/item.tsx)) to display data for a single record.

Overriding a model route with a custom page only requires to provide an appropriately named file within the pages folder. Because in Next.js predefined routes take precedence over dynamic routes, all requests for that model will now point to the new page.

In the example below, a custom `books.tsx` page is overriding the default `/books` route that would be otherwise provided by `[model]/index.tsx`.

```
pages
├── [model]
│   ├── index.tsx
│   └── [item].tsx
├── books.tsx
├── index.tsx
└── login.tsx
```

### Next.js Resources

To learn more about Next.js, take a look at the following resources:

- [Next.js Documentation](https://nextjs.org/docs) - learn about Next.js features and API.
- [Learn Next.js](https://nextjs.org/learn) - an interactive Next.js tutorial.
