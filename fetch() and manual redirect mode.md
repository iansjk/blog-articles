---
title: fetch() and manual redirect mode
created: 2024-08-24
tags:
  - "#web"
---
Recently at work I needed to make a request to an endpoint in the browser that I knew would return a `302 Found` response. The subsequent redirected response would return a 4xx error, but it could be safely disregarded.

The desired behavior was to resolve the request if the endpoint returned a 302 (disregarding the subsequent 4xx error), and reject otherwise (e.g. if the initial request to the endpoint instead returned a 4xx or 5xx code).

We typically use axios at work, but just using `axios.post` on the endpoint would cause it to always reject because the redirected response had a 4xx status code.[^1] Ideally we could disregard the redirect and then resolve the Promise based on the `302 Found` response.

Unfortunately, older versions of axios in browsers [don't allow you to disable redirects](https://github.com/axios/axios/issues/3924); the reason is that axios uses XMLHttpRequest under the hood, which always follows redirects. (There is a [`fetch`-based adapter](https://github.com/axios/axios/issues/1219) that allows redirect behavior to be configured, but it's unfortunately only supported in axios 1.7.x+, and we haven't yet migrated to axios 1.x+.)

Based on the above I ended up reaching for `fetch()` with [`redirect: 'manual'`](https://fetch.spec.whatwg.org/#concept-request-redirect-mode), which seemed like it'd do what I wanted. At first I had written something like
```ts
const res = await fetch("/endpoint", {
  method: "POST",
  redirect: "manual"
});

if (res.statusCode >= 200 && res.statusCode < 400) {
  // looks like what we expect, continue...
}
```
But this doesn't actually work. There's a subtle pitfall: when using `fetch()` with `redirect: 'manual'`, subsequent redirects return what is called an [opaque-redirect filtered response](https://fetch.spec.whatwg.org/#concept-filtered-response-opaque-redirect), which has some specific properties:
- Response type of `opaqueredirect`
- Status code `0`
- Empty status message, headers, and body

Importantly, since the status code is 0 for such a response, the above conditional won't return true even when the initial request gave us a 302 as expected. 

Thankfully, the solution is straightforward: if the initial request to the endpoint returns an error (and not the expected `302 Found`),
- `res.statusCode` will reflect that (e.g. `500`)
- `res.type` won't be `opaqueredirect`, but something else (e.g. `'cors'`)

So in this case, we need to amend our conditional to:
```ts
if (res.type === 'opaqueredirect') {
  // looks like what we expect, continue...
}
```

As for *why* `fetch()` with `redirect: 'manual'` returns an opaque-redirect filtered response: [HTTP redirects are not exposed to APIs in the Fetch spec](https://fetch.spec.whatwg.org/#atomic-http-redirect-handling). The reason is security: 
> A fetch to `https://example.org/auth` that includes a `Cookie` marked `HttpOnly` could result in a redirect to `https://other-origin.invalid/4af955781ea1c84a3b11`. This new URL contains a secret. If we expose redirects that secret would be available through a cross-site scripting attack.

Note that all of this is apparently different from how `fetch` works in Node environments; for example, [`node-fetch` returns a basic filtered response instead of the opaque-redirect filtered response](https://www.npmjs.com/package/node-fetch#manual-redirect), and I'm not sure how the new global fetch that became stable in Node 21 differs (if at all) in redirect behavior from the browser. But that'll be a topic for another time.

[^1]: Another option would have been to use a custom [validateStatus](https://axios-http.com/docs/handling_errors) instead of the default that rejects for non-2xx/3xx codes, but I would rather not follow the subsequent redirect at all...