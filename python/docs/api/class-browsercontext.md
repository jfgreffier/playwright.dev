---
id: class-browsercontext
title: "BrowserContext"
---

* extends: [EventEmitter]

BrowserContexts provide a way to operate multiple independent browser sessions.

If a page opens another page, e.g. with a `window.open` call, the popup will belong to the parent page's browser context.

Playwright allows creation of "incognito" browser contexts with `browser.newContext()` method. "Incognito" browser contexts don't write any browsing data to disk.

```py
# async

# create a new incognito browser context
context = await browser.new_context()
# create a new page inside context.
page = await context.new_page()
await page.goto("https://example.com")
# dispose context once it"s no longer needed.
await context.close()
```

```py
# sync

# create a new incognito browser context
context = browser.new_context()
# create a new page inside context.
page = context.new_page()
page.goto("https://example.com")
# dispose context once it"s no longer needed.
context.close()
```


- [browser_context.on("close")](./api/class-browsercontext.md#browser_contextonclose)
- [browser_context.on("page")](./api/class-browsercontext.md#browser_contextonpage)
- [browser_context.add_cookies(cookies)](./api/class-browsercontext.md#browser_contextadd_cookiescookies)
- [browser_context.add_init_script(**options)](./api/class-browsercontext.md#browser_contextadd_init_scriptoptions)
- [browser_context.browser](./api/class-browsercontext.md#browser_contextbrowser)
- [browser_context.clear_cookies()](./api/class-browsercontext.md#browser_contextclear_cookies)
- [browser_context.clear_permissions()](./api/class-browsercontext.md#browser_contextclear_permissions)
- [browser_context.close()](./api/class-browsercontext.md#browser_contextclose)
- [browser_context.cookies(**options)](./api/class-browsercontext.md#browser_contextcookiesoptions)
- [browser_context.expect_event(event, **options)](./api/class-browsercontext.md#browser_contextexpect_eventevent-options)
- [browser_context.expect_page(**options)](./api/class-browsercontext.md#browser_contextexpect_pageoptions)
- [browser_context.expose_binding(name, callback, **options)](./api/class-browsercontext.md#browser_contextexpose_bindingname-callback-options)
- [browser_context.expose_function(name, callback)](./api/class-browsercontext.md#browser_contextexpose_functionname-callback)
- [browser_context.grant_permissions(permissions, **options)](./api/class-browsercontext.md#browser_contextgrant_permissionspermissions-options)
- [browser_context.new_page()](./api/class-browsercontext.md#browser_contextnew_page)
- [browser_context.pages](./api/class-browsercontext.md#browser_contextpages)
- [browser_context.route(url, handler)](./api/class-browsercontext.md#browser_contextrouteurl-handler)
- [browser_context.set_default_navigation_timeout(timeout)](./api/class-browsercontext.md#browser_contextset_default_navigation_timeouttimeout)
- [browser_context.set_default_timeout(timeout)](./api/class-browsercontext.md#browser_contextset_default_timeouttimeout)
- [browser_context.set_extra_http_headers(headers)](./api/class-browsercontext.md#browser_contextset_extra_http_headersheaders)
- [browser_context.set_geolocation(geolocation)](./api/class-browsercontext.md#browser_contextset_geolocationgeolocation)
- [browser_context.set_offline(offline)](./api/class-browsercontext.md#browser_contextset_offlineoffline)
- [browser_context.storage_state(**options)](./api/class-browsercontext.md#browser_contextstorage_stateoptions)
- [browser_context.unroute(url, **options)](./api/class-browsercontext.md#browser_contextunrouteurl-options)
- [browser_context.wait_for_event(event, **options)](./api/class-browsercontext.md#browser_contextwait_for_eventevent-options)

## browser_context.on("close")

Emitted when Browser context gets closed. This might happen because of one of the following:
* Browser context is closed.
* Browser application is closed or crashed.
* The [browser.close()](./api/class-browser.md#browserclose) method was called.

## browser_context.on("page")
- type: <[Page]>

The event is emitted when a new Page is created in the BrowserContext. The page may still be loading. The event will also fire for popup pages. See also [page.on("popup")](./api/class-page.md#pageonpopup) to receive events about popups relevant to a specific page.

The earliest moment that page is available is when it has navigated to the initial url. For example, when opening a popup with `window.open('http://example.com')`, this event will fire when the network request to "http://example.com" is done and its response has started loading in the popup.

```py
# async

async with context.expect_page() as page_info:
    await page.click("a[target=_blank]"),
page = await page_info.value
print(await page.evaluate("location.href"))
```

```py
# sync

with context.expect_page() as page_info:
    page.click("a[target=_blank]"),
page = page_info.value
print(page.evaluate("location.href"))
```

:::note
Use [page.wait_for_load_state(**options)](./api/class-page.md#pagewait_for_load_stateoptions) to wait until the page gets to a particular state (you should not need it in most cases).
:::

## browser_context.add_cookies(cookies)
- `cookies` <[List]\[[Dict]\]>
  - `name` <[str]>
  - `value` <[str]>
  - `url` <[str]> either url or domain / path are required. Optional.
  - `domain` <[str]> either url or domain / path are required Optional.
  - `path` <[str]> either url or domain / path are required Optional.
  - `expires` <[float]> Unix time in seconds. Optional.
  - `httpOnly` <[bool]> Optional.
  - `secure` <[bool]> Optional.
  - `sameSite` <"Strict"|"Lax"|"None"> Optional.

Adds cookies into this browser context. All pages within this context will have these cookies installed. Cookies can be obtained via [browser_context.cookies(**options)](./api/class-browsercontext.md#browser_contextcookiesoptions).

```py
# async

await browser_context.add_cookies([cookie_object1, cookie_object2])
```

```py
# sync

browser_context.add_cookies([cookie_object1, cookie_object2])
```

## browser_context.add_init_script(**options)
- `path` <[Union]\[[str], [pathlib.Path]\]> Path to the JavaScript file. If `path` is a relative path, then it is resolved relative to the current working directory. Optional.
- `script` <[str]> Script to be evaluated in all pages in the browser context. Optional.

Adds a script which would be evaluated in one of the following scenarios:
* Whenever a page is created in the browser context or is navigated.
* Whenever a child frame is attached or navigated in any page in the browser context. In this case, the script is evaluated in the context of the newly attached frame.

The script is evaluated after the document was created but before any of its scripts were run. This is useful to amend the JavaScript environment, e.g. to seed `Math.random`.

An example of overriding `Math.random` before the page loads:

```js
// preload.js
Math.random = () => 42;
```

```py
# async

# in your playwright script, assuming the preload.js file is in same directory.
await browser_context.add_init_script(path="preload.js")
```

```py
# sync

# in your playwright script, assuming the preload.js file is in same directory.
browser_context.add_init_script(path="preload.js")
```

:::note
The order of evaluation of multiple scripts installed via [browser_context.add_init_script(**options)](./api/class-browsercontext.md#browser_contextadd_init_scriptoptions) and [page.add_init_script(**options)](./api/class-page.md#pageadd_init_scriptoptions) is not defined.
:::

## browser_context.browser
- returns: <[NoneType]|[Browser]>

Returns the browser instance of the context. If it was launched as a persistent context null gets returned.

## browser_context.clear_cookies()

Clears context cookies.

## browser_context.clear_permissions()

Clears all permission overrides for the browser context.

```py
# async

context = await browser.new_context()
await context.grant_permissions(["clipboard-read"])
# do stuff ..
context.clear_permissions()
```

```py
# sync

context = browser.new_context()
context.grant_permissions(["clipboard-read"])
# do stuff ..
context.clear_permissions()
```

## browser_context.close()

Closes the browser context. All the pages that belong to the browser context will be closed.

:::note
The default browser context cannot be closed.
:::

## browser_context.cookies(**options)
- `urls` <[str]|[List]\[[str]\]> Optional list of URLs.
- returns: <[List]\[[Dict]\]>
  - `name` <[str]>
  - `value` <[str]>
  - `domain` <[str]>
  - `path` <[str]>
  - `expires` <[float]> Unix time in seconds.
  - `httpOnly` <[bool]>
  - `secure` <[bool]>
  - `sameSite` <"Strict"|"Lax"|"None">

If no URLs are specified, this method returns all cookies. If URLs are specified, only cookies that affect those URLs are returned.

## browser_context.expect_event(event, **options)
- `event` <[str]> Event name, same one typically passed into `*.on(event)`.
- `predicate` <[Callable]> Receives the event data and resolves to truthy value when the waiting should resolve.
- `timeout` <[float]> Maximum time to wait for in milliseconds. Defaults to `30000` (30 seconds). Pass `0` to disable timeout. The default value can be changed by using the [browser_context.set_default_timeout(timeout)](./api/class-browsercontext.md#browser_contextset_default_timeouttimeout).
- returns: <[EventContextManager]>

Performs action and waits for given `event` to fire. If predicate is provided, it passes event's value into the `predicate` function and waits for `predicate(event)` to return a truthy value. Will throw an error if browser context is closed before the `event` is fired.

```py
# async

async with context.expect_event("page") as event_info:
    await context.click("button")
page = await event_info.value
```

```py
# sync

with context.expect_event("page") as event_info:
    context.click("button")
page = event_info.value
```

## browser_context.expect_page(**options)
- `predicate` <[Callable]\[[Page]\]:[bool]> Receives the [Page] object and resolves to truthy value when the waiting should resolve.
- `timeout` <[float]> Maximum time to wait for in milliseconds. Defaults to `30000` (30 seconds). Pass `0` to disable timeout. The default value can be changed by using the [browser_context.set_default_timeout(timeout)](./api/class-browsercontext.md#browser_contextset_default_timeouttimeout).
- returns: <[EventContextManager]\[[Page]\]>

Performs action and waits for `page` event to fire. If predicate is provided, it passes [Page] value into the `predicate` function and waits for `predicate(event)` to return a truthy value. Will throw an error if the page is closed before the worker event is fired.

## browser_context.expose_binding(name, callback, **options)
- `name` <[str]> Name of the function on the window object.
- `callback` <[Callable]> Callback function that will be called in the Playwright's context.
- `handle` <[bool]> Whether to pass the argument as a handle, instead of passing by value. When passing a handle, only one argument is supported. When passing by value, multiple arguments are supported.

The method adds a function called `name` on the `window` object of every frame in every page in the context. When called, the function executes `callback` and returns a [Promise] which resolves to the return value of `callback`. If the `callback` returns a [Promise], it will be awaited.

The first argument of the `callback` function contains information about the caller: `{ browserContext: BrowserContext, page: Page, frame: Frame }`.

See [page.expose_binding(name, callback, **options)](./api/class-page.md#pageexpose_bindingname-callback-options) for page-only version.

An example of exposing page URL to all frames in all pages in the context:

```py
# async

import asyncio
from playwright.async_api import async_playwright

async def run(playwright):
    webkit = playwright.webkit
    browser = await webkit.launch(headless=false)
    context = await browser.new_context()
    await context.expose_binding("pageURL", lambda source: source["page"].url)
    page = await context.new_page()
    await page.set_content("""
    <script>
      async function onClick() {
        document.querySelector('div').textContent = await window.pageURL();
      }
    </script>
    <button onclick="onClick()">Click me</button>
    <div></div>
    """)
    await page.click("button")

async def main():
    async with async_playwright() as playwright:
        await run(playwright)
asyncio.run(main())
```

```py
# sync

from playwright.sync_api import sync_playwright

def run(playwright):
    webkit = playwright.webkit
    browser = webkit.launch(headless=false)
    context = browser.new_context()
    context.expose_binding("pageURL", lambda source: source["page"].url)
    page = context.new_page()
    page.set_content("""
    <script>
      async function onClick() {
        document.querySelector('div').textContent = await window.pageURL();
      }
    </script>
    <button onclick="onClick()">Click me</button>
    <div></div>
    """)
    page.click("button")

with sync_playwright() as playwright:
    run(playwright)
```

An example of passing an element handle:

```py
# async

async def print(source, element):
    print(await element.text_content())

await context.expose_binding("clicked", print, handle=true)
await page.set_content("""
  <script>
    document.addEventListener('click', event => window.clicked(event.target));
  </script>
  <div>Click me</div>
  <div>Or click me</div>
""")
```

```py
# sync

def print(source, element):
    print(element.text_content())

context.expose_binding("clicked", print, handle=true)
page.set_content("""
  <script>
    document.addEventListener('click', event => window.clicked(event.target));
  </script>
  <div>Click me</div>
  <div>Or click me</div>
""")
```

## browser_context.expose_function(name, callback)
- `name` <[str]> Name of the function on the window object.
- `callback` <[Callable]> Callback function that will be called in the Playwright's context.

The method adds a function called `name` on the `window` object of every frame in every page in the context. When called, the function executes `callback` and returns a [Promise] which resolves to the return value of `callback`.

If the `callback` returns a [Promise], it will be awaited.

See [page.expose_function(name, callback)](./api/class-page.md#pageexpose_functionname-callback) for page-only version.

An example of adding an `md5` function to all pages in the context:

```py
# async

import asyncio
import hashlib
from playwright.async_api import async_playwright

async def sha1(text):
    m = hashlib.sha1()
    m.update(bytes(text, "utf8"))
    return m.hexdigest()


async def run(playwright):
    webkit = playwright.webkit
    browser = await webkit.launch(headless=False)
    context = await browser.new_context()
    await context.expose_function("sha1", sha1)
    page = await context.new_page()
    await page.set_content("""
        <script>
          async function onClick() {
            document.querySelector('div').textContent = await window.sha1('PLAYWRIGHT');
          }
        </script>
        <button onclick="onClick()">Click me</button>
        <div></div>
    """)
    await page.click("button")

async def main():
    async with async_playwright() as playwright:
        await run(playwright)
asyncio.run(main())
```

```py
# sync

import hashlib
from playwright.sync_api import sync_playwright

def sha1(text):
    m = hashlib.sha1()
    m.update(bytes(text, "utf8"))
    return m.hexdigest()


def run(playwright):
    webkit = playwright.webkit
    browser = webkit.launch(headless=False)
    context = browser.new_context()
    context.expose_function("sha1", sha1)
    page = context.new_page()
    page.expose_function("sha1", sha1)
    page.set_content("""
        <script>
          async function onClick() {
            document.querySelector('div').textContent = await window.sha1('PLAYWRIGHT');
          }
        </script>
        <button onclick="onClick()">Click me</button>
        <div></div>
    """)
    page.click("button")

with sync_playwright() as playwright:
    run(playwright)
```

## browser_context.grant_permissions(permissions, **options)
- `permissions` <[List]\[[str]\]> A permission or an array of permissions to grant. Permissions can be one of the following values:
  * `'geolocation'`
  * `'midi'`
  * `'midi-sysex'` (system-exclusive midi)
  * `'notifications'`
  * `'push'`
  * `'camera'`
  * `'microphone'`
  * `'background-sync'`
  * `'ambient-light-sensor'`
  * `'accelerometer'`
  * `'gyroscope'`
  * `'magnetometer'`
  * `'accessibility-events'`
  * `'clipboard-read'`
  * `'clipboard-write'`
  * `'payment-handler'`
- `origin` <[str]> The [origin] to grant permissions to, e.g. "https://example.com".

Grants specified permissions to the browser context. Only grants corresponding permissions to the given origin if specified.

## browser_context.new_page()
- returns: <[Page]>

Creates a new page in the browser context.

## browser_context.pages
- returns: <[List]\[[Page]\]>

Returns all open pages in the context. Non visible pages, such as `"background_page"`, will not be listed here. You can find them using [chromium_browser_context.background_pages](./api/class-chromiumbrowsercontext.md#chromium_browser_contextbackground_pages).

## browser_context.route(url, handler)
- `url` <[str]|[Pattern]|[Callable]\[[URL]\]:[bool]> A glob pattern, regex pattern or predicate receiving [URL] to match while routing.
- `handler` <[Callable]\[[Route], [Request]\]> handler function to route the request.

Routing provides the capability to modify network requests that are made by any page in the browser context. Once route is enabled, every request matching the url pattern will stall unless it's continued, fulfilled or aborted.

An example of a naïve handler that aborts all image requests:

```py
# async

context = await browser.new_context()
page = await context.new_page()
await context.route("**/*.{png,jpg,jpeg}", lambda route: route.abort())
await page.goto("https://example.com")
await browser.close()
```

```py
# sync

context = browser.new_context()
page = context.new_page()
context.route("**/*.{png,jpg,jpeg}", lambda route: route.abort())
page.goto("https://example.com")
browser.close()
```

or the same snippet using a regex pattern instead:

```py
# async

context = await browser.new_context()
page = await context.new_page()
await context.route(r"(\.png$)|(\.jpg$)", lambda route: route.abort())
page = await context.new_page()
await page.goto("https://example.com")
await browser.close()
```

```py
# sync

context = browser.new_context()
page = context.new_page()
context.route(r"(\.png$)|(\.jpg$)", lambda route: route.abort())
page = await context.new_page()
page = context.new_page()
page.goto("https://example.com")
browser.close()
```

Page routes (set up with [page.route(url, handler)](./api/class-page.md#pagerouteurl-handler)) take precedence over browser context routes when request matches both handlers.

:::note
Enabling routing disables http cache.
:::

## browser_context.set_default_navigation_timeout(timeout)
- `timeout` <[float]> Maximum navigation time in milliseconds

This setting will change the default maximum navigation time for the following methods and related shortcuts:
* [page.go_back(**options)](./api/class-page.md#pagego_backoptions)
* [page.go_forward(**options)](./api/class-page.md#pagego_forwardoptions)
* [page.goto(url, **options)](./api/class-page.md#pagegotourl-options)
* [page.reload(**options)](./api/class-page.md#pagereloadoptions)
* [page.set_content(html, **options)](./api/class-page.md#pageset_contenthtml-options)
* [page.wait_for_navigation(**options)](./api/class-page.md#pagewait_for_navigationoptions)

:::note
[page.set_default_navigation_timeout(timeout)](./api/class-page.md#pageset_default_navigation_timeouttimeout) and [page.set_default_timeout(timeout)](./api/class-page.md#pageset_default_timeouttimeout) take priority over [browser_context.set_default_navigation_timeout(timeout)](./api/class-browsercontext.md#browser_contextset_default_navigation_timeouttimeout).
:::

## browser_context.set_default_timeout(timeout)
- `timeout` <[float]> Maximum time in milliseconds

This setting will change the default maximum time for all the methods accepting `timeout` option.

:::note
[page.set_default_navigation_timeout(timeout)](./api/class-page.md#pageset_default_navigation_timeouttimeout), [page.set_default_timeout(timeout)](./api/class-page.md#pageset_default_timeouttimeout) and [browser_context.set_default_navigation_timeout(timeout)](./api/class-browsercontext.md#browser_contextset_default_navigation_timeouttimeout) take priority over [browser_context.set_default_timeout(timeout)](./api/class-browsercontext.md#browser_contextset_default_timeouttimeout).
:::

## browser_context.set_extra_http_headers(headers)
- `headers` <[Dict]\[[str], [str]\]> An object containing additional HTTP headers to be sent with every request. All header values must be strings.

The extra HTTP headers will be sent with every request initiated by any page in the context. These headers are merged with page-specific extra HTTP headers set with [page.set_extra_http_headers(headers)](./api/class-page.md#pageset_extra_http_headersheaders). If page overrides a particular header, page-specific header value will be used instead of the browser context header value.

:::note
[browser_context.set_extra_http_headers(headers)](./api/class-browsercontext.md#browser_contextset_extra_http_headersheaders) does not guarantee the order of headers in the outgoing requests.
:::

## browser_context.set_geolocation(geolocation)
- `geolocation` <[NoneType]|[Dict]>
  - `latitude` <[float]> Latitude between -90 and 90.
  - `longitude` <[float]> Longitude between -180 and 180.
  - `accuracy` <[float]> Non-negative accuracy value. Defaults to `0`.

Sets the context's geolocation. Passing `null` or `undefined` emulates position unavailable.

```py
# async

await browser_context.set_geolocation({"latitude": 59.95, "longitude": 30.31667})
```

```py
# sync

browser_context.set_geolocation({"latitude": 59.95, "longitude": 30.31667})
```

:::note
Consider using [browser_context.grant_permissions(permissions, **options)](./api/class-browsercontext.md#browser_contextgrant_permissionspermissions-options) to grant permissions for the browser context pages to read its geolocation.
:::

## browser_context.set_offline(offline)
- `offline` <[bool]> Whether to emulate network being offline for the browser context.

## browser_context.storage_state(**options)
- `path` <[Union]\[[str], [pathlib.Path]\]> The file path to save the storage state to. If `path` is a relative path, then it is resolved relative to [current working directory](https://nodejs.org/api/process.html#process_process_cwd). If no path is provided, storage state is still returned, but won't be saved to the disk.
- returns: <[Dict]>
  - `cookies` <[List]\[[Dict]\]>
    - `name` <[str]>
    - `value` <[str]>
    - `domain` <[str]>
    - `path` <[str]>
    - `expires` <[float]> Unix time in seconds.
    - `httpOnly` <[bool]>
    - `secure` <[bool]>
    - `sameSite` <"Strict"|"Lax"|"None">
  - `origins` <[List]\[[Dict]\]>
    - `origin` <[str]>
    - `localStorage` <[List]\[[Dict]\]>
      - `name` <[str]>
      - `value` <[str]>

Returns storage state for this browser context, contains current cookies and local storage snapshot.

## browser_context.unroute(url, **options)
- `url` <[str]|[Pattern]|[Callable]\[[URL]\]:[bool]> A glob pattern, regex pattern or predicate receiving [URL] used to register a routing with [browser_context.route(url, handler)](./api/class-browsercontext.md#browser_contextrouteurl-handler).
- `handler` <[Callable]\[[Route], [Request]\]> Optional handler function used to register a routing with [browser_context.route(url, handler)](./api/class-browsercontext.md#browser_contextrouteurl-handler).

Removes a route created with [browser_context.route(url, handler)](./api/class-browsercontext.md#browser_contextrouteurl-handler). When `handler` is not specified, removes all routes for the `url`.

## browser_context.wait_for_event(event, **options)
- `event` <[str]> Event name, same one would pass into `browserContext.on(event)`.
- `predicate` <[Callable]> Receives the event data and resolves to truthy value when the waiting should resolve.
- `timeout` <[float]> Maximum time to wait for in milliseconds. Defaults to `30000` (30 seconds). Pass `0` to disable timeout. The default value can be changed by using the [browser_context.set_default_timeout(timeout)](./api/class-browsercontext.md#browser_contextset_default_timeouttimeout).
- returns: <[Any]>

Waits for event to fire and passes its value into the predicate function. Returns when the predicate returns truthy value. Will throw an error if the context closes before the event is fired. Returns the event data value.

```py
# async

context = await browser.new_context()
await context.grant_permissions(["geolocation"])
```

```py
# sync

context = browser.new_context()
context.grant_permissions(["geolocation"])
```


[Accessibility]: ./api/class-accessibility.md "Accessibility"
[Browser]: ./api/class-browser.md "Browser"
[BrowserContext]: ./api/class-browsercontext.md "BrowserContext"
[BrowserType]: ./api/class-browsertype.md "BrowserType"
[CDPSession]: ./api/class-cdpsession.md "CDPSession"
[ChromiumBrowserContext]: ./api/class-chromiumbrowsercontext.md "ChromiumBrowserContext"
[ConsoleMessage]: ./api/class-consolemessage.md "ConsoleMessage"
[Dialog]: ./api/class-dialog.md "Dialog"
[Download]: ./api/class-download.md "Download"
[ElementHandle]: ./api/class-elementhandle.md "ElementHandle"
[FileChooser]: ./api/class-filechooser.md "FileChooser"
[Frame]: ./api/class-frame.md "Frame"
[JSHandle]: ./api/class-jshandle.md "JSHandle"
[Keyboard]: ./api/class-keyboard.md "Keyboard"
[Mouse]: ./api/class-mouse.md "Mouse"
[Page]: ./api/class-page.md "Page"
[Playwright]: ./api/class-playwright.md "Playwright"
[Request]: ./api/class-request.md "Request"
[Response]: ./api/class-response.md "Response"
[Route]: ./api/class-route.md "Route"
[Selectors]: ./api/class-selectors.md "Selectors"
[TimeoutError]: ./api/class-timeouterror.md "TimeoutError"
[Touchscreen]: ./api/class-touchscreen.md "Touchscreen"
[Video]: ./api/class-video.md "Video"
[WebSocket]: ./api/class-websocket.md "WebSocket"
[Worker]: ./api/class-worker.md "Worker"
[Element]: https://developer.mozilla.org/en-US/docs/Web/API/element "Element"
[Evaluation Argument]: ./core-concepts.md#evaluationargument "Evaluation Argument"
[Promise]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise "Promise"
[iterator]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols "Iterator"
[origin]: https://developer.mozilla.org/en-US/docs/Glossary/Origin "Origin"
[selector]: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors "selector"
[Serializable]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify#Description "Serializable"
[UIEvent.detail]: https://developer.mozilla.org/en-US/docs/Web/API/UIEvent/detail "UIEvent.detail"
[UnixTime]: https://en.wikipedia.org/wiki/Unix_time "Unix Time"
[xpath]: https://developer.mozilla.org/en-US/docs/Web/XPath "xpath"

[Any]: https://docs.python.org/3/library/typing.html#typing.Any "Any"
[bool]: https://docs.python.org/3/library/stdtypes.html "bool"
[Callable]: https://docs.python.org/3/library/typing.html#typing.Callable "Callable"
[EventContextManager]: https://docs.python.org/3/reference/datamodel.html#context-managers "Event context manager"
[EventEmitter]: https://pyee.readthedocs.io/en/latest/#pyee.BaseEventEmitter "EventEmitter"
[Dict]: https://docs.python.org/3/library/typing.html#typing.Dict "Dict"
[float]: https://docs.python.org/3/library/stdtypes.html#numeric-types-int-float-complex "float"
[int]: https://docs.python.org/3/library/stdtypes.html#numeric-types-int-float-complex "int"
[List]: https://docs.python.org/3/library/typing.html#typing.List "List"
[NoneType]: https://docs.python.org/3/library/constants.html#None "None"
[Pattern]: https://docs.python.org/3/library/re.html "Pattern"
[URL]: https://en.wikipedia.org/wiki/URL "URL"
[pathlib.Path]: https://realpython.com/python-pathlib/ "pathlib.Path"
[str]: https://docs.python.org/3/library/stdtypes.html#text-sequence-type-str "str"
[Union]: https://docs.python.org/3/library/typing.html#typing.Union "Union"