# Example: Parallel Code Review Session

This walkthrough shows a real 4-model council session for security-critical code. You will see how each model catches different issues, and why the combined output is far stronger than any single review.


## Scenario

You are about to push authentication middleware for a Node.js/Express application. The middleware handles JWT verification, session management, and role-based access control. It sits in front of every protected route.

This is security-critical code. A bug here means unauthorized access, leaked user data, or both. A full 4-model council is warranted.


## The Code Under Review

```javascript
// middleware/auth.js
const jwt = require('jsonwebtoken');
const redis = require('../config/redis');
const User = require('../models/User');

const JWT_SECRET = process.env.JWT_SECRET;

module.exports = function authenticate(requiredRole) {
  return async (req, res, next) => {
    try {
      // Extract token from Authorization header
      const authHeader = req.headers.authorization;
      if (!authHeader) {
        return res.status(401).json({ error: 'No authorization header' });
      }

      const token = authHeader.split(' ')[1];
      if (!token) {
        return res.status(401).json({ error: 'No token provided' });
      }

      // Verify JWT
      const decoded = jwt.verify(token, JWT_SECRET);

      // Check session in Redis
      const session = await redis.get(`session:${decoded.userId}`);
      if (!session) {
        return res.status(401).json({ error: 'Session expired' });
      }
      const sessionData = JSON.parse(session);

      // Look up user and check role
      const user = await User.findById(decoded.userId);
      if (!user) {
        return res.status(401).json({ error: 'User not found' });
      }

      if (requiredRole && user.role !== requiredRole) {
        return res.status(403).json({ error: 'Insufficient permissions' });
      }

      // Attach user to request
      req.user = user;
      req.session = sessionData;
      next();
    } catch (err) {
      if (err.name === 'TokenExpiredError') {
        return res.status(401).json({ error: 'Token expired' });
      }
      if (err.name === 'JsonWebTokenError') {
        return res.status(401).json({ error: 'Invalid token' });
      }
      return res.status(500).json({ error: 'Authentication failed' });
    }
  };
};
```

The code looks clean. It works. Most single-reviewer passes would approve it with minor comments. But watch what happens when four models review it in parallel.


## The Prompt

Send this to all four models simultaneously:

```
You are reviewing security-critical Express middleware before it ships to
production. The middleware handles JWT auth, session management, and
role-based access control for a Node.js API serving ~50k requests/minute.

Review the code below for:
1. Security vulnerabilities (prioritize these)
2. Edge cases that could cause unexpected behavior
3. Performance concerns at the stated scale
4. Maintainability and testability

Be specific. Cite line numbers. Suggest fixes, not just problems.

[paste the code above]
```


## Model Responses

Each model found real issues. Almost no overlap between them. This is the Complementary Fragments pattern at work.


### Claude's Review

**Focus: Architecture and information leakage**

> The middleware has a tight coupling problem that will bite you in testing and make refactoring painful later.
>
> **Direct imports create hard dependencies.** Lines 2-3 import the Redis client and the User model directly. You cannot unit test this middleware without standing up Redis and a database (or setting up elaborate mocks that mirror implementation details). Extract these into injectable dependencies:
>
> ```javascript
> // Instead of this:
> const redis = require('../config/redis');
> const User = require('../models/User');
>
> // Accept them as configuration:
> module.exports = function createAuthMiddleware({ sessionStore, userLookup }) {
> ```
>
> **Error responses leak information to attackers.** The catch block on lines 48-54 returns different messages for "Token expired" vs. "Invalid token." This distinction helps an attacker determine whether a stolen token is merely expired (worth trying to refresh) or entirely invalid (move on to a different attack). Return a single generic message for all auth failures:
>
> ```javascript
> return res.status(401).json({ error: 'Authentication failed' });
> ```
>
> **Overall:** Good logic flow, poor separation of concerns. The happy path is correct, but the architecture makes this hard to test and the error handling helps attackers more than it helps developers.


### GPT-4's Review

**Focus: Edge cases and robustness**

> **Token refresh race condition (lines 25-28).** There is a window between `jwt.verify()` succeeding and the Redis session lookup. If the token expires during that window (unlikely but possible under load), the user gets a "Session expired" error when the real problem is token expiry. At 50k req/min, this will happen. Add a grace period:
>
> ```javascript
> const decoded = jwt.verify(token, JWT_SECRET, {
>   clockTolerance: 5  // 5-second grace period
> });
> ```
>
> **Role comparison is fragile (line 38).** The check `user.role !== requiredRole` uses bare string comparison. One typo in a route definition (`"adnin"` instead of `"admin"`) silently blocks all access with no error in the logs. Define roles as an enum or a constants object:
>
> ```javascript
> const ROLES = Object.freeze({
>   ADMIN: 'admin',
>   USER: 'user',
>   MODERATOR: 'moderator',
> });
> ```
>
> **No request correlation for debugging.** When auth failures spike, you need to trace them. Attach a request ID to every auth error response so you can correlate failures across services.
>
> **Overall:** Functional but fragile at the edges. The happy path works; the failure modes need tightening.


### Gemini's Review

**Focus: Performance at scale**

> **Two network round trips on every request (lines 28 and 34).** The middleware hits Redis, then hits the database, on every single request. At 50k req/min, that is 50k Redis calls and 50k DB queries per minute just for auth. This is your bottleneck.
>
> **Recommended fix:** Cache the combined session + user data in memory with a short TTL. A 30-second in-memory cache reduces your Redis calls by ~97% for active users (who make multiple requests within 30 seconds):
>
> ```javascript
> const NodeCache = require('node-cache');
> const authCache = new NodeCache({ stdTTL: 30, checkperiod: 10 });
>
> // Inside the middleware:
> const cacheKey = `auth:${decoded.userId}`;
> let cached = authCache.get(cacheKey);
> if (!cached) {
>   const session = await sessionStore.get(`session:${decoded.userId}`);
>   const user = await userLookup(decoded.userId);
>   cached = { session: JSON.parse(session), user };
>   authCache.set(cacheKey, cached);
> }
> ```
>
> **Latency estimates:**
> - Current: 15-25ms per request (Redis RTT + DB query)
> - With in-memory cache: 2-5ms for cache hits, 15-25ms for cache misses
> - At 50k req/min with 30s TTL: ~97% cache hit rate for active users
>
> **Missing security headers (not in this middleware, but related).** Authenticated responses should include `Cache-Control: no-store` to prevent proxies and browsers from caching sensitive data. Consider adding it here or in a companion middleware.
>
> **Overall:** Correct but suboptimal for high-traffic routes. The performance gap is significant at the stated scale.


### Grok's Review

**Focus: Security hardening and attack vectors**

> **Algorithm confusion vulnerability (line 25).** `jwt.verify(token, JWT_SECRET)` does not specify which algorithms to accept. An attacker can craft a token with `"alg": "none"` and bypass verification entirely. Always pin your algorithms:
>
> ```javascript
> const decoded = jwt.verify(token, JWT_SECRET, {
>   algorithms: ['HS256'],
> });
> ```
>
> This is a well-known attack (CVE-2015-9235) and many JWT libraries are vulnerable if you do not explicitly restrict algorithms.
>
> **Header parsing is unsafe (line 18).** `authHeader.split(' ')[1]` assumes the header follows the exact format `Bearer <token>`. A malformed header with extra spaces (e.g., `Bearer  token` or `Bearer token with spaces`) produces unexpected results. Validate the format:
>
> ```javascript
> const parts = authHeader.split(' ');
> if (parts.length !== 2 || parts[0] !== 'Bearer') {
>   return res.status(401).json({ error: 'Malformed authorization header' });
> }
> const token = parts[1];
> ```
>
> **Insufficient logging for security monitoring (lines 48-54).** Failed auth attempts are swallowed into generic HTTP responses with no server-side logging. You need IP address, user agent, timestamp, and the failure reason for your security team:
>
> ```javascript
> logger.warn('Auth failure', {
>   reason: err.name,
>   ip: req.ip,
>   userAgent: req.headers['user-agent'],
>   path: req.originalUrl,
>   timestamp: new Date().toISOString(),
> });
> ```
>
> **Rate limiting question.** This middleware does not rate-limit failed auth attempts. An attacker can brute-force tokens at full speed. Either add rate limiting here or confirm it is handled upstream (e.g., at the API gateway or reverse proxy).
>
> **Overall:** The logic works, but the security posture needs hardening. The algorithm confusion issue is the most urgent finding.


## Merged Review Output

Here is the reconciled, prioritized list of findings across all four models:

| Priority | Category | Finding | Found by |
|----------|----------|---------|----------|
| **P0** | Security | Algorithm confusion vulnerability; `jwt.verify` accepts `alg: "none"` | Grok |
| **P0** | Security | Error responses leak token state to attackers | Claude |
| **P1** | Security | Header parsing breaks on malformed Authorization values | Grok |
| **P1** | Security | No server-side logging of auth failures (IP, user agent, path) | Grok |
| **P2** | Architecture | Tight coupling to Redis client and User model; not unit-testable | Claude |
| **P3** | Performance | Two network round trips per request; add in-memory caching layer | Gemini |
| **P4** | Robustness | Token refresh race condition under load; add clock tolerance | GPT-4 |
| **P4** | Robustness | Role check uses bare strings; typos fail silently | GPT-4 |
| **P5** | Headers | Missing `Cache-Control: no-store` on authenticated responses | Gemini |

Nine distinct findings. No single model caught more than four.


## What a Single Model Would Have Missed

Consider what happens if you only use one model:

- **Claude alone** catches architecture and info-leakage issues but misses the algorithm confusion vulnerability (P0) and the performance bottleneck (P3).
- **GPT-4 alone** finds the edge cases but misses every security vulnerability Grok identified and the performance issues Gemini flagged.
- **Gemini alone** nails the performance analysis but misses all five security and robustness findings from the other models.
- **Grok alone** hardens the security posture but overlooks the architectural debt that will slow down every future change to this middleware.

The combined review is not incrementally better. It is categorically more thorough. Each model brought a lens the others lacked. That is the [Complementary Fragments](../methodology/03_reconciliation_patterns.md) pattern in its purest form.


## The Fixed Code

Here is the middleware after incorporating all P0 through P2 fixes:

```javascript
// middleware/auth.js
const jwt = require('jsonwebtoken');
const NodeCache = require('node-cache');
const logger = require('../config/logger');

const JWT_SECRET = process.env.JWT_SECRET;
const ALGORITHMS = ['HS256'];
const AUTH_CACHE_TTL = 30; // seconds

const authCache = new NodeCache({ stdTTL: AUTH_CACHE_TTL, checkperiod: 10 });

const ROLES = Object.freeze({
  ADMIN: 'admin',
  USER: 'user',
  MODERATOR: 'moderator',
});

module.exports = function createAuthMiddleware({ sessionStore, userLookup }) {
  return function authenticate(requiredRole) {
    return async (req, res, next) => {
      try {
        // Extract and validate Authorization header format
        const authHeader = req.headers.authorization;
        if (!authHeader) {
          return res.status(401).json({ error: 'Authentication failed' });
        }

        const parts = authHeader.split(' ');
        if (parts.length !== 2 || parts[0] !== 'Bearer') {
          return res.status(401).json({ error: 'Authentication failed' });
        }
        const token = parts[1];

        // Verify JWT with pinned algorithm and clock tolerance
        const decoded = jwt.verify(token, JWT_SECRET, {
          algorithms: ALGORITHMS,
          clockTolerance: 5,
        });

        // Check session and user (with in-memory cache)
        const cacheKey = `auth:${decoded.userId}`;
        let cached = authCache.get(cacheKey);

        if (!cached) {
          const session = await sessionStore.get(`session:${decoded.userId}`);
          if (!session) {
            logAuthFailure(req, 'session_not_found');
            return res.status(401).json({ error: 'Authentication failed' });
          }

          const user = await userLookup(decoded.userId);
          if (!user) {
            logAuthFailure(req, 'user_not_found');
            return res.status(401).json({ error: 'Authentication failed' });
          }

          cached = { session: JSON.parse(session), user };
          authCache.set(cacheKey, cached);
        }

        // Check role
        if (requiredRole && cached.user.role !== requiredRole) {
          logAuthFailure(req, 'insufficient_role');
          return res.status(403).json({ error: 'Insufficient permissions' });
        }

        req.user = cached.user;
        req.session = cached.session;
        next();
      } catch (err) {
        logAuthFailure(req, err.name || 'unknown_error');
        return res.status(401).json({ error: 'Authentication failed' });
      }
    };
  };
};

function logAuthFailure(req, reason) {
  logger.warn('Auth failure', {
    reason,
    ip: req.ip,
    userAgent: req.headers['user-agent'],
    path: req.originalUrl,
    timestamp: new Date().toISOString(),
  });
}

module.exports.ROLES = ROLES;
```

**What changed:**

- **Algorithm pinned** to HS256; the `alg: "none"` attack is blocked.
- **Error messages unified**; all auth failures return the same generic message.
- **Header parsing validates format**; rejects malformed Authorization values.
- **Auth failures logged** with IP, user agent, path, and reason.
- **Dependencies injected** via `sessionStore` and `userLookup`; the middleware is now unit-testable with simple stubs.
- **In-memory cache** added; cuts redundant Redis and DB calls at scale.
- **Clock tolerance** added; handles the token-expiry race condition.
- **Roles exported** as a frozen constant object for use in route definitions.

The P5 fix (Cache-Control header) was left for a companion middleware, since it applies to all authenticated responses, not just this middleware's scope.


## Takeaway

Four models, one prompt, nine findings, zero overlap in their top concerns. The entire council session takes about 60 seconds of wall-clock time (all four calls run in parallel) and catches issues that would have required four separate specialists to find manually.

When the code is security-critical, run the full council. The cost of four API calls is trivial compared to the cost of shipping an algorithm confusion vulnerability to production.
