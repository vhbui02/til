# Cookies

A small piece of data a server sends to a user's web browser.

The browser can
- Store cookies.
- Create new cookies.
- Modify existing ones.
- Send them back to the same server.

Cookies is the answer to HTTP statelessness by design.

## Use cases

In general: The serve use the contents of HTTP cookies to identify which request is from which browser/user, then issue a personalized/generic response:

- Session management: User sign-in status, shopping cart contents, game scores, ... that server needs to remember
- Personalization: user pref such as display language and UI theme
- Tracking: recording and analyze user behavior

Cookies in a user authentication system:

1. Client sends sign-in credentials (via form submission, Postman, ...) to Server.
1. Server checks the credentials. If correct, Server sends back new HTML (if SSR) or JSON (if CSR) + a cookie containing a session ID that records their sign-in status.
1. User changes page => Client sends the cookie with session ID along with the request, right now Client still thinks the user is signed in, or the session ID is still valid.
1. Server checks the session ID:
  - If valid, sends the user a personalized version of the new page.
  - If not valid, session ID is deleted, and the feedback is varied: user can be shown a generic version of the page, or an access denied message and redirect to log in page, or send a refresh tokens.

## Data Storage

Historically speaking, cookies were used for general client-side data storage. There is a limit however:

- Hundreds of cookies/domain.
- 4KB/cookie.

Cookies are sent with **every** request, if the number and size of the cookies is significant enough they can worsen performance.

Nowadays, browser WebAPI supports Web Storage API (`localStorage`/`sessionStorage`) and `IndexedDB`. They can store data up to 4MB.

> [!CAUTION]
>
> Web Storage API is vulnerable to XSS attack. **DO NOT** use it to store sensitive information such as JWT.

## Syntax

HTTP Response:

```
HTTP/1.1 200 OK
Content-Type: application/json
// ...
Set-Cookie: <name>=<value>[; <Max-Age>=<age>]
`[; expires=<date>][; domain=<domain_name>]
[; SameSite=[Strict,Lax,None]]
[; Path=<some_path>][; Secure][; HttpOnly]
Set-Cookie: ...
```

Subsequent HTTP Request:

```
GET /foo.html HTTP/1.1
Host: www.example.com
Cookie: <name>=<value>; <name>=<value>
// ...
```

### `Max-Age=` and `Expires=`

Based on whether `Max-Age=` or `Expires=` are set that people categorized cookies into 3 types:

- **Permanent cookies:** cookies that are deleted after the date specified in the `Expires=` attribute or after the period specified in `Max-Age=` attribute. `Expires=` is more error-prone due to the possibility of expiration time mismatched setting between client and server, therefore `Max-Age=` should be used all the time. 

> [!TIPS]
>
> Cookies that are used to store sensitive information (e.g. JWT) should have a short lifetime.

- **Session cookies:** cookies without a `Max-Age=` or `Expires=` attribute are deleted with the session ends. Browser defines when the session ends, usually when all the browser tabs and windows are closed. Some browsers implements *session restoring* which revive the session cookies, thus make it last indefinately.

> [!CAUTION]
>
> Prevent session fixation attacks by regenerating and resend session cookies whenever there is an auth request even when there is one still existed.

- **Zombie cookies:** Cookies that're revived after being deleted.

### `Secure` and `HttpOnly`

Cookies values are accessible by end user and can be stolen if not carefully configured.

```
Set-Cookie: id=abcdef; Max-Age=3600; Secure; HttpOnly
```

=> Cookie can only be sent with an encrypted request ever HTTPS protocol and can't be accessed by JavaScript. Therefore avoiding MITM attack, XSS attack and filesystem unauthorized access.

### `Domain=`

`Domain=` specifies the scope of a cookie—that is, the domains and subdomains to which the cookie is allowed to be sent. A server can only set `Domain=` attribute to **its own domain or the parent domain**, not to a subdomain or some other domain.
```
Set-Cookie: id=a3fWa; Expires=Thu, 21 Oct 2021 07:28:00 GMT; Secure; HttpOnly; Domain=mozilla.org
```

=> Cookie are available to `mozilla.org` and subdomains `developer.mozilla.org`, `bugzilla.mozilla.org`, ...

Without `Domain=`, the cookies are only available on the server that sets it. This mitigate session fixation attack (an attacker that gain access to one of the compromised subdomains can hijack a Cookie from parent domain)

However, one can forget about this and set `Domain=`, browser has implemented some *cookie prefixes* - a set of naming conventions that enables asserting facts about the cookie that the browser received

- `__Host-<cookie-name>`: Domain-locked cookie
  - Have `Secure;` attribute.
  - Send from an origin whose schema is `https://`.
  - Do not include `Domain=` attribute.
  - Have `Path=/` attribute.

- `__Secure-`: weaker than `__Host-`
  - Have `Secure;` attribute.
  - Send from an origin whose schema is `https://`.

**Implementation:** Top-level domain `example.com` set a cookie named `__Host-sessionid` whose attributes reflect the above rules. On the server, the web application **MUST check for the full cookie name**, including the prefix.

If an attacker tries to set a subdomain-created cookie named `__Host-sessionid` from `compromised.example.com` subdomain with `Domain=example.com` hoping to capture cookie value from `example.com`, the browser will ignore it because the `__Host-` prefix requires the cookie to be set from the host domain without a `Domain=` attribute.

### `Path=`

`Path=` indicates a URL path **must exist in the requested URL**, in order for the Client to send the `Cookie` header.

```
Set-Cookie: id=a3fWa; Expires=Thu, 21 Oct 2021 07:28:00 GMT; Secure; HttpOnly; Path=/docs
```

=> Matched API endpoints starts with `/docs`, `/docs/`, `/docs/Web`, ... 

### `SameSite=`

Lets servers specify whether/when cookies are sent with cross-site requests.

- `SameSite=Strict`: browser only send cookie in response to requests originating from the cookie's origin site.

- `SameSite=Lax`: browser also sends the cookie when the user navigates to the cookie's origin site when user is coming from a different site.
  - You visit `example.com` directly in your browser => Cookie sent
  - You are on `other.com` visit `example.com` by clicking an anchor tag (top-level navigation) => Cookie sent
  - You are on `other.com`, send a POST request to `example.com` via `<form>` elements or JS. => Cookie sent
  - You are on `other.com`, display an image from `example.com` (cross-site subresource request) => Cookie NOT sent.

- `SameSite=None; Secure`: browser sends cookies on both originating and cross-site requests.
  - Client visits `https://myapp.com` 
  - `myapp.com` redirects user to `https://auth.example.com/login`
  - Client sends a POST request to an API endpoint whose domain is `auth.example.com`.
  - Server whose domain is `auth.example.com` sends an HTTP Response with a cookie as follow:

    ```
    Set-Cookie: sessionId=abc123; SameSite=None; Secure; Domain=example.com; Path=/; HttpOnly
    ```
  
  - Next time, when Client from `myapp.com` makes a request to `https://auth.example.com/api/userinfo`, the browser includes `sessionid` cookie.


## Server-side cookies updating

Application can use programming languages's built-in API to send a HTTP Response with a `Set-Cookie:` header with the existing name and a new value.

Reason: User updates their references and the application wants to reflect this changes client-side.

## Client-side cookies operations

Browser JavaScript runtime environment implements `Document.cookie` and `Cookie Store API` to modify cookie data.

```js
// create new cookie
document.cookie = "foo=bar";
document.cookie = "alpha=a";
document.cookie = "beta=b";

// modify existing cookies
// NOTE: only cookies without HttpOnly attribute
document.cookie = "foo=baz";
console.log(document.cookie); // output: "alpha=a; beta=b; foo=baz"
```

**NOTE:** modifying Cookies client-side then send it using `Cookie:` HTTP Request Header WON'T change the server-side data. It's best to forbid it from the start by setting `HttpOnly` attribute.

## Security




## Reference

- [MDN Web Docs "Guides > Using HTTP Cookies"](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies)
- [HttpOnly Cookies](https://owasp.org/www-community/HttpOnly)