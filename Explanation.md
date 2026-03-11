# Bug Explanation

## 1. What was the bug?

The `HttpClient.request()` method failed to refresh the OAuth2 token when `oauth2Token` was a plain JavaScript object instead of an `OAuth2Token` instance. This resulted in API requests being sent without an Authorization header, causing authentication failures.

**Specific scenario:** When `oauth2Token = { accessToken: "stale", expiresAt: 0 }` (a plain object), the code didn't recognize it as invalid and didn't refresh it, leaving `headers.Authorization` as `undefined`.

## 2. Why did it happen?

The original conditional logic had a flaw in handling different token states:

```typescript
// Original buggy code
if (
  !this.oauth2Token ||
  (this.oauth2Token instanceof OAuth2Token && this.oauth2Token.expired)
) {
  this.refreshOAuth2();
}
```

This condition only refreshed the token when:
- Token was falsy (`null` or `undefined`), OR
- Token was an `OAuth2Token` instance AND expired

**The bug:** When `oauth2Token` was a plain object (truthy but not an `OAuth2Token` instance), the condition evaluated to `false`, so `refreshOAuth2()` was never called. The subsequent check `if (this.oauth2Token instanceof OAuth2Token)` also failed, so no Authorization header was set.

This can occur in real applications when:
- Token data is deserialized from JSON storage/cache
- Token is received from an API as a plain object
- Type information is lost during data transfer between systems

## 3. Why does your fix solve it?

The fix restructures the logic to check all three invalid token states explicitly:

```typescript
// Fixed code
if (
  !this.oauth2Token ||
  !(this.oauth2Token instanceof OAuth2Token) ||
  this.oauth2Token.expired
) {
  this.refreshOAuth2();
}
```

Now the token is refreshed when ANY of these conditions are true:
- Token is falsy (null/undefined)
- Token is NOT an OAuth2Token instance (plain object)
- Token is an OAuth2Token instance but expired

This ensures that plain objects are caught by the `!(this.oauth2Token instanceof OAuth2Token)` check, triggering a refresh. After refresh, `oauth2Token` is guaranteed to be a valid `OAuth2Token` instance, so the Authorization header is set correctly.

## 4. What's one realistic case / edge case your tests still don't cover?

**Uncovered edge case: Concurrent request race conditions**

The current tests don't cover the scenario where multiple API requests are made simultaneously while the token is expired or invalid. 

**The problem:**
```typescript
// Request 1 starts: token is expired
// refreshOAuth2() is called, but takes time

// Request 2 starts immediately after: token still expired
// refreshOAuth2() is called AGAIN (unnecessary)

// Both requests might use different token values
// Or cause multiple unnecessary refresh API calls
```

**Why this matters in production:**
- Multiple concurrent requests could trigger redundant token refreshes
- Race conditions could lead to using stale tokens between refresh start and completion
- Backend OAuth2 providers might rate-limit or reject concurrent refresh requests

**Potential solution (not implemented to keep changes minimal):**
- Add a `refreshPromise` property to track in-flight refreshes
- Make `refreshOAuth2()` return a Promise
- Reuse the same promise for concurrent requests
- Implement a mutex/lock pattern for token refresh operations

This would require more substantial changes beyond the minimal fix scope, but is a realistic consideration for production systems handling high-throughput API traffic.
