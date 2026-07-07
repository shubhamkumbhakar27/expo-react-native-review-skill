# Rule 11 — Offline & secrets safety

**Penalty −200/occurrence · Reward +50**

The React Query cache is **persisted** (`gcTime: Infinity` on persisted queries) and **purged on logout**. Volatile/sensitive domains (chat/message bodies, tokens) are **excluded from persistence**. Auth tokens live in **secure storage** (`expo-secure-store` / Keychain), never `AsyncStorage` or the query cache. Offline is treated as *authenticated-but-offline* — connectivity/timeout errors must never clear the session; only a real 401 on the auth probe may.

## Why it matters
Losing the cache on cold start means blank screens offline. Persisting secrets or storing tokens in plain AsyncStorage leaks them. Clearing the session on a timeout logs users out every time the network blips.

## Detection
- `AsyncStorage.setItem("token", ...)` / token stored in query cache or Redux.
- Persister with no dehydrate filter → chat/PII written to disk.
- Logout path that doesn't `queryClient.clear()` / purge the persister.
- `catch` on a request that calls `signOut()` for any error (not just 401).
- Persisted queries without `gcTime: Infinity` (evicted, defeating persistence).

## Bad
```ts
await AsyncStorage.setItem("accessToken", token);   // secret in plain storage
onError: () => signOut(),                            // logs out on any error incl. timeout
```

## Good
```ts
import * as SecureStore from "expo-secure-store";
await SecureStore.setItemAsync("accessToken", token);        // secure store
// only a real 401 from the auth probe clears the session:
if (isAuthProbe && err.status === 401) signOut();
// persister excludes volatile domains:
dehydrateOptions: { shouldDehydrateQuery: q => q.queryKey[0] !== "chat" }
// logout:
await persister.removeClient(); queryClient.clear();
```

## Scoring
- −200 per secret in plain storage, un-purged cache on logout, persisted secret/PII, or session cleared on non-401 error.
- +50 for secure token storage + logout purge + persistence filter in the diff.

## Edge cases
- On web, secure store falls back — ensure the platform branch degrades safely.
- Optimistic offline mutations should queue/retry on reconnect, not error out permanently.
