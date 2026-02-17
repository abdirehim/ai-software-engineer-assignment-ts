# Bug Fix Explanation

## What was the bug?

The `HttpClient.request()` method failed to refresh OAuth2 tokens when they were stored as plain objects (e.g., `{accessToken: "stale", expiresAt: 0}`). The code only refreshed tokens that were `null` or expired `OAuth2Token` instances, but ignored plain object tokens entirely.

## Why did it happen?

The original condition used `instanceof OAuth2Token` to check token validity:
```typescript
if (!this.oauth2Token || (this.oauth2Token instanceof OAuth2Token && this.oauth2Token.expired))
```

This logic has a gap: when `oauth2Token` is a plain object, `instanceof OAuth2Token` returns `false`, so the entire condition evaluates to `false` and the token is never refreshed.

## Why does your fix solve it?

The fix adds a check for non-OAuth2Token instances:
```typescript
if (
  !this.oauth2Token ||
  !(this.oauth2Token instanceof OAuth2Token) ||
  this.oauth2Token.expired
)
```

Now any token that isn't an `OAuth2Token` instance gets refreshed, handling plain objects correctly.

## Uncovered edge case

The tests don't cover **concurrent token refresh scenarios**. If multiple requests trigger refresh simultaneously, race conditions could cause duplicate refresh calls or inconsistent token state across requests.
