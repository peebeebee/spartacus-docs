---
title: Server-Side Rendering Coding Guidelines
---

The following guidelines are highly recommended when working with server-side rendering (SSR).

***

**Table of Contents**

- This will become a table of contents (this text will be scrapped).
{:toc}

***

## Working with Global Objects

Do not access global objects that are available in the browser. For example, do not use the `window`, `document`, `navigator`, and other browser types, because they do not exist on the server. If you try to use them, or any library that uses them, it will not work. For most cases, it is better to inject `WindowRef` and then do additional checks. For example, you can check if `WindowRef.nativeWindow` is defined.

## Working with Timeouts

Limit or avoid using `setTimeout`. It slows down the server-side rendering process and should be removed from the `ngOnDestroy` method of your components.

For RxJs timeouts, cancel their stream on success, because they can slow down rendering as well.

## Manipulating the nativeElement

Do not manipulate the `nativeElement` directly. Instead, use `Renderer2` and related methods. We do this to ensure that, in any environment, we are able to change our view. The following is an example:

```typescript
constructor(element: ElementRef, renderer: Renderer2) {
  renderer.setStyle(element.nativeElement, 'font-size', 'x-large');
}
```

## Using Transfer State Functionality

Using transfer state functionality is recommended. The application runs XHR requests on the server, and then again on the client-side, when the application bootstraps.

Use a cache that is transferred from the server to the client.

For more information, see [Configurable State Persistence and Rehydration]({{ site.baseurl }}{% link _pages/dev/configurable-state-persistence-and-rehydration.md %}).

## Getting the Request URL and Origin

In CCv2, or any other setup that uses proxy servers, the request origin may be modified on the fly to something else, such as `localhost` or `127.0.0.1`. Also, the `document.location` behaves differently in the browser as compared to SSR, where Angular Universal creates a DOM that is more limited in functionality.

When working with SSR, to get the request URL or origin, you should use the Spartacus `SERVER_REQUEST_URL` and `SERVER_REQUEST_ORIGIN` injection tokens. The following is an example:

```ts
constructor(
  /* ... */
  @Optional() @Inject(SERVER_REQUEST_URL) protected serverRequestUrl: string | null,
  @Optional() @Inject(SERVER_REQUEST_ORIGIN) protected serverRequestOrigin: string | null
)
```

**Note:** The `@Optional()` decorator is necessary. If it is not included, the injection will crash for client-side rendering (CSR) because these tokens are not provided in CSR.

### Workaround for Known Issue in Spartacus 3.0.2 and Earlier

In Spartacus 3.0.2 and earlier, when using CCv2 or any other setup that uses proxy servers, the `SERVER_REQUEST_URL` and `SERVER_REQUEST_ORIGIN` tokens return the `localhost` request origin instead of the real website domain. This has been fixed in Spartacus 3.0.3.

If you are using Spartacus 3.0.2 or earlier, and you are unable to upgrade to the latest version of Spartacus, you can fix this issue with the workaround described in the following steps.

1. Copy the following code, preferably to a separate file alongside `app.server.module.ts`:

    ```ts
    import { StaticProvider } from '@angular/core';
    import { REQUEST } from '@nguniversal/express-engine/   tokens';
    import { Request } from 'express';
    import { SERVER_REQUEST_ORIGIN, SERVER_REQUEST_URL } from     '@spartacus/core';

    export const fixServerRequestProviders: StaticProvider[]    = [
      {
        provide: SERVER_REQUEST_URL,
        useFactory: getRequestUrl,
        deps: [REQUEST],
      },
      {
        provide: SERVER_REQUEST_ORIGIN,
        useFactory: getRequestOrigin,
        deps: [REQUEST],
      },
    ];

    function getRequestUrl(req: Request): string {
      return getRequestOrigin(req) + req.originalUrl;
    }

    function getRequestOrigin(req: Request): string {
      return req.protocol + '://' + req.hostname;
    }
    ```

2. Add `fixServerRequestProviders` to the providers array in `app.server.module.ts `, as follows:

    ```ts
    providers: [
      // ...
      ...fixServerRequestProviders
    ],
    ```

3. Enable the `trust proxy` option in `server.ts`, below the line containing `const server = express();`, as follows:

    ```ts
      server.set('trust proxy', 'loopback');
    ```  
