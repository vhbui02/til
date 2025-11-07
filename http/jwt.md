# JWT

## Overview

JWT (JSON Web Token) is an _encoded_ (NOTE: not _encrypted_) message that stores "claims" (a set of key-value pairs):

- Who issued the token?
- Who is the token issued to?
- Who is the intended recipient of the token?
- Expiration date.
- ...

JWT syntax message is composed of 3 components, delimited by dot notation `.`:

`<header>.<payload>.<signature>`

Header (after decoded) JSON structure:

```json
{
  "alg": "HS256", // signature of the encrypted algorithm
  "typ": "JWT" // type of token
}
```

Payload (after decoded) JSON structure:

```json
{
  "fresh": false, // fresh tokeness pattern
  "iat": 1754025416, // issued at
  "jti": "b602bb40-1bf5-400a-bc97-f05758375874", // JWT ID - unique identifier of this token
  "type": "access", // access token
  // "type": "refresh" // refresh token
  "sub": "e84e76439eba17189cad1979d700b1c3", // subject claims - whom the token refers to
  "nbf": 1754025416, // not valid before
  "csrf": "8f72c4ed-c229-42ce-978d-63d65c5501f2", // CSRF token of this token
  "exp": 1754026316 // expiration time
}
```

Signature = `HMACSHA256(base64Url(header) + "." + base64Url(payload), secretKey)`
=> Without the secret key, JWT can't be modified under radar.

## Use cases

- **Authorization** in general.
- Secure data exchange between front-end and back-end.
- **Stateless:** no need server-side session anymore.

## JWT Validation and JWT Verification

- **Validation:** make sure JWT components has the correct claims. Does header have "alg", "typ"? Does payload have "iat", "jti", ...
- **Verification:** make sure JWT components hasn't been tampered with. After recalculating signature at server-side does it still match?

## `Authorization: Bearer <token>`

JWT Bearer authentication scheme: specify how token should be read from `Authorization` Header and check its validity.

The scheme is configured at your server storing resources. The token's validity is checked using a symmetrical key or an asymmetrical key (included in a certificate file)

## JWT management

When a client receives a JWT, it must store and use the token securely to prevent unauthorized access and attacks.

1. ALWAYS store the access token and refresh token inside an `HttpOnly;` cookie, avoid storing token in `localStorage` or `sessionStorage` to prevent access from JavaScript, mitigate XSS risks.

2. Use the _double submit token pattern_ for CSRF protection with either form-based or header-based validation:

- **Form-based:** Render the CSRF token inside a hidden input field using server-side templates.
- **Header-based:** Store the CSRF token in a non-`HttpOnly` cookie. This allows JavaScript to read the token and include it in a custom header (e.g., `X-CSRF-TOKEN`) for API requests. When using `fetch()` with `credentials: 'include'`, cookies matching the domain and attributes will be sent automatically.

CSRF attack relies on the attacker tricking a user accessing a malicious site and from there, sending an authenticated request to the target site. By design, cookies are sent automatically by browsers with requests to the same origin, regardless of **WHERE** the request originated. However, due to Browser's `Same-Origin Policy` documents or scripts loaded from one origin CAN NOT interact with resources from another origin, the malicious site's code can't read the CSRF token storing inside the 2nd cookie, therefore it can't set `X-CSRF-TOKEN`.

If the API is cross-origin, configure CORS on the server to allow credentials and the required headers.

After receiving the token, server will compare the value in the custom header with the CSRF token stored inside JWT payload (under "csrf" key).

**Example:** If your cookies are set with `SameSite=Strict; Domain=example.com`, and your frontend and backend are on subdomains, such as `api.example.com` or `static.example.com`, make sure to configure the server allowing **credentials** and **origins** by setting header `Access-Control-Allow-Credentials: true` and `Access-Control-Allow-Origin: <insert_domain>` (NOT `*`) or use a CORS library (e.g. `Flask-CORS`)

3. Always send the JWT with requests to protected endpoints, either automatically via cookies or manually in the `Authorization` header if required by the API.

4. Remove and invalidate the JWT upon user logout or session expiration to prevent reuse.

5. Add `Secure;` attribute to enforce HTTPS-only requests/responses.

**TIPS:** If the use case allows, set `SameSite=Secure` for both can avoid the cookies from being included inside the authoritative request from the malicious site during a CSRF attack.

#### Refresh Token

Refresh Token is a long-lived JWT that can only be used to create new access tokens.
There are 2 ways to utilize:

- **Simple (Frontend-dependent):** store access token's expires time on front-end, each time a request is made, check if the current access token is near or already expired, then refresh it as needed.
- **Complex (Frontend-agnostic):** make a dedicated request to a specific API endpoint with the current access token, then check the result to see if it's worked. If not, use the refresh token to generate a new access token and resend the request.

**NOTE:** Refresh Token is recommended when frontend is NOT a website (e.g. Mobile, API-only, ...)

**CAUTION:** If an Access Token is invalidated (e.g. a user is logged out, user's account is deleted from database, ...), corresponding Refresh Tokens must be revoked also.

## Token Freshness Pattern

An access token received right after an authentication request is deemed "fresh". After some time, this token should no longer by considered "fresh". Some sensitive routes requiring fresh JWT will block user access if the token is not fresh.

Use case:

- Email/Password changing route.
