# fusion-plugin-csrf-protection

[![Build status](https://badge.buildkite.com/3fef89529147193838107b8bf6a5e0cb9f1dc8d11502461920.svg?branch=master)](https://buildkite.com/uberopensource/fusion-plugin-csrf-protection)

Adds CSRF protection to requests that use non-idempotent HTTP methods.

This package provides a modified `fetch` that is automatically secure against CSRF attacks.

If you're making requests to CSRF protected endpoints from React, you should use [fusion-plugin-csrf-protection-react](https://github.com/fusionjs/fusion-plugin-csrf-protection-react) instead of this package.

### Installation

```sh
yarn add fusion-plugin-csrf-protection
```

### Example

```js
// src/main.js
import React from 'react';
import {FetchToken, SessionToken} from 'fusion-tokens';
import App from 'fusion-react';
import Session from 'fusion-plugin-jwt';
import CsrfProtection, {FetchForCsrfToken} from 'fusion-plugin-csrf-protection';
import fetch from unfetch;

export default () => {
  const app = new App(<div></div>);

  app.register(SessionToken, Session);
  app.register(FetchForCsrfToken, fetch);
  app.register(FetchToken, CsrfProtection);

  if (__BROWSER__) {
    app.register(FetchForCsrfToken, fetch);
    app.middleware({fetch: FetchToken}, ({fetch}) => {
      // makes a pre-flight request for CSRF token if required,
      // and prevents POST calls to /api/hello without a valid token
      const res = await fetch('/api/hello', {method: 'POST'});
    });
  }
  else {
    app.middleware((ctx, next) => {
      if (ctx.method === 'POST' && ctx.path === '/api/hello') {
        ctx.body = {hello: 'world'};
      }
      return next();
    });
  }
}
```

### API

#### Dependency registration

```js
import CsrfProtection, {FetchForCsrfToken} from 'fusion-plugin-csrf-protection';
import {FetchToken, SessionToken} from 'fusion-tokens';

app.register(SessionToken, Session);
app.register(FetchForCsrfToken, fetch);
app.register(FetchToken, CsrfProtection);
```

The `fusion-plugin-csrf-protection` module provides an api that matches the `fetch` api,
and therefore can be registered on the standard `FetchToken` exported by `fusion-tokens`.

* `fetch: (url: string, options: Object) => Promise` - A [`fetch`](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) implementation.
* `Session` - a Session plugin, such as the one provided by [`fusion-plugin-jwt`](https://github.com/fusionjs/fusion-plugin-jwt).
  The Session instance should expose a `get: (key: string) => string` and `set: (key: string, value: string) => string` methods.

#### Instance API

```js
if (__BROWSER__) {
  app.middleware({fetch: FetchToken}, ({fetch}) => {
    // makes a pre-flight request for CSRF token if required,
    // and prevents POST calls to /api/hello without a valid token
    const res = await fetch('/hello/world', {method: 'POST'});
  });
}
```

`fetch: (url: string, options: Object) => Promise` - Client-only. A decorated `fetch` function that automatically does pre-flight requests for CSRF tokens if required.
