# GraphQL-based Project Memo

## Integrated Apollo-Client

### Step - 1: Link

- Create HttpLink

```typescript
const httpLink = new HttpLink({
	uri: SERVER_URL,
});
```

- Create authLink

```typescript
const authMiddleware = setContext(async (_, { headers, ...context }) => {
  let token = '';
  if (isAuthenticated) {
    token = await getAccessTokenSilently({
      audience: 'https://assistant.cmsk-icool.com/api',
    });
  }
  return {
    headers: {
      ...headers,
      ...(token ? { Authorization: `Bearer ${token}` } : {}),
    },
  };
});
```

- Create errorLink

```typescript
const errorLink = onError(({ graphQLErrors, networkError }) => {
  if (graphQLErrors) {
    graphQLErrors.forEach(({ message, locations, path }) =>
      console.error(
        `[GraphQL error]: Message: ${message}, Location: ${JSON.stringify(
          locations,
          null,
          2
        )}, Path: ${path}`
      )
    );
  }

  if (networkError) {
    console.error(`[Network error]: ${networkError}`);
  }
});
```

- Compose above links by `ApolloLink.from`

```typescript
const link = ApolloLink.from([errorLink, authMiddleware, httpLink]);
```

