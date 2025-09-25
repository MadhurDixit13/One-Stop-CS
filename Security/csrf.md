# Cross-Site Request Forgery (CSRF)

CSRF is an attack where a victim’s browser sends unintended, state-changing requests to a site where the user is authenticated. Because browsers automatically attach cookies, a malicious site can trick the browser into performing actions as the user unless protections are in place.

---

## Why It Works

- Implicit credentials: cookies are sent automatically on cross-site requests
- State-changing endpoints without anti-CSRF checks
- Missing validation of request origin (no token, no Origin/Referer checks)

---

## Attack Flow

1. User logs into `bank.com` (session cookie set).
2. User visits `evil.com` which silently submits a form to `bank.com/transfer`.
3. Browser includes `bank.com` cookies with that request.
4. Without CSRF defenses, the transfer executes as the user.

### Minimal attack (auto-submitting form)
```html
<!-- evil.com -->
<form action="https://bank.com/transfer" method="POST">
  <input type="hidden" name="to" value="attacker" />
  <input type="hidden" name="amount" value="1000" />
</form>
<script>document.forms[0].submit();</script>
```

### GET misuse (should never mutate state via GET)
```html
<img src="https://bank.com/delete-account?id=123" />
```

---

## Core Defenses (layer them)

1. SameSite cookies
   - Lax (recommended default): blocks cookies on cross-site POSTs; allows top-level GET navigations
   - Strict: blocks cookies on all cross-site requests
   - Example: `Set-Cookie: session=abc; Path=/; HttpOnly; Secure; SameSite=Lax`
2. CSRF tokens (synchronizer token pattern)
   - Unpredictable token tied to session/user; include in forms/headers and verify server-side
3. Double-submit cookie
   - Send the same token in a cookie and a header/body and compare
4. Validate Origin/Referer headers on state-changing requests
5. Require custom headers (e.g., `X-Requested-With`) and JSON-only APIs
6. Re-authenticate or step-up for critical actions
7. Never mutate state on GET; enforce checks on POST/PUT/PATCH/DELETE

Notes:
- CORS does not stop CSRF by itself; it controls cross-origin reads, not cookie sending
- CAPTCHAs are not a CSRF control (may slow attackers, not prevent CSRF)

---

## Framework Snippets

### Express/Node (`csurf`)
```js
import express from 'express';
import cookieParser from 'cookie-parser';
import csurf from 'csurf';

const app = express();
app.use(cookieParser());
app.use(express.urlencoded({ extended: false }));
app.use(express.json());

const csrfProtection = csurf({ cookie: { httpOnly: true, sameSite: 'lax', secure: true } });

app.get('/form', csrfProtection, (req, res) => {
  res.send(`<form method="POST" action="/transfer">
    <input type="hidden" name="_csrf" value="${req.csrfToken()}" />
    <input name="to" />
    <input name="amount" />
    <button>Send</button>
  </form>`);
});

app.post('/transfer', csrfProtection, (req, res) => {
  res.send('ok'); // invalid token -> 403
});
```

### Django
```python
# settings.py
CSRF_TRUSTED_ORIGINS = ['https://yourdomain.com']
CSRF_COOKIE_SAMESITE = 'Lax'
CSRF_COOKIE_SECURE = True
SESSION_COOKIE_SECURE = True
# In templates: use {% csrf_token %}
```

### Spring Security
```java
// CSRF is enabled by default for browser clients; ensure forms include token
http
  .csrf(csrf -> csrf
    .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
  );
```

---

## SPAs (React/Vue/Angular)

If using cookies for auth, send credentials intentionally and include a CSRF token header.
```js
fetch('/api/transfer', {
  method: 'POST',
  credentials: 'include',
  headers: {
    'Content-Type': 'application/json',
    'X-CSRF-Token': token
  },
  body: JSON.stringify({ to, amount })
});
```
Guidelines:
- Store the CSRF token in memory or a non-HttpOnly cookie used only for the header
- Prefer `SameSite=Lax` session cookies plus server-side token verification
- Do not expose tokens to third-party origins

Double-submit cookie pattern (stateless APIs):
```js
// Server sets: Set-Cookie: csrf=RANDOM; SameSite=Lax; Secure
fetch('/api/action', {
  method: 'POST',
  credentials: 'include',
  headers: { 'X-CSRF-Token': getCookie('csrf') }
});
```

---

## Origin/Referer Validation

Check `Origin` first, then fallback to `Referer`. Only allow your own origins.
```js
function verifyOrigin(req, res, next) {
  const allowed = new Set(['https://yourdomain.com']);
  const origin = req.headers.origin || '';
  const referer = req.headers.referer || '';
  const ok = [...allowed].some(a => origin.startsWith(a) || referer.startsWith(a));
  if (!ok) return res.status(403).send('Forbidden');
  next();
}
```

---

## Common Pitfalls

- Relying solely on CORS
- Mutating state on GET
- Not setting `SameSite`/`Secure`/`HttpOnly` on cookies
- Forgetting token validation on JSON APIs

---

## Testing Your Defenses

- Attempt cross-origin form submissions and expect 403
- Alter/miss CSRF token and verify rejection
- Inspect cookies for `Secure`, `HttpOnly`, `SameSite`
- Confirm Origin/Referer checks on POST/PUT/PATCH/DELETE

---

## References

- OWASP CSRF Prevention Cheat Sheet: https://owasp.org/www-community/attacks/csrf
- MDN SameSite cookies: https://developer.mozilla.org/docs/Web/HTTP/Headers/Set-Cookie/SameSite
- OWASP Testing Guide (CSRF): https://owasp.org/www-project-web-security-testing-guide/
