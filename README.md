# pwa-training

PWA Training with my personal notes and labs.

## Intro

Progressive Web Apps (PWAs) help developers to provide native-app qualities in web applications that are reliable, fast, and engaging.

PWA is not an API or a technology, but it is a web development approach that uses a combination of tools and technologies to create ideal user experiences.

## Understand your audience and content

How do you start thinking about building an app or a website?

You should understand your target and audience:

- Audience
- Platforms
- Connectivity
- Data cost
- Usage contexts

See slides at: https://docs.google.com/presentation/d/154nnHaw5kgj9fwMu8PQA6nr0f6nZmzwwr0Oq-0j6GfY

## Design for All Your Users

Design means something more than just graphic or visual design. “Design” in the context of PWAs means taking a mobile-first approach when building your app, with an emphasis on accessibility.

## Core Technologies

Most API’s in the PWA space are built on JavaScript Promises and the Fetch API.

ES2015 functions syntax:

```
// ES5
var func = function(x,y) {
  return x + y;
}
// ES2015
var func = (x, y) => {
  return x + y;
};
```

Fetch API example:

```
fetch('/examples/example.json')
.then(response => {
  return response.json();
}) // .then do something with the data
.catch(error => {
  console.log('Fetch failed', error);
});
```

## Intro to Service Workers


Service workers are at the core of PWA techniques for resource-caching and push notifications.

It's a client-side programmable proxy between your webapp and the outside world. It executes separately of the web browser thread.

It's a kind of Web Worker.

About the *Web Workers* concept: https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API

Web Workers makes it possible to run a script operation in a background thread separate from the main execution thread of a web application. The advantage of this is that laborious processing can be performed in a separate thread, allowing the main (usually the UI) thread to run without being blocked/slowed down.

Because workers run separately from the main thread, service workers are independent of the application they are associated with. This has several consequences:

  - Because the service worker is not blocking (it's designed to be fully asynchronous) synchronous XHR and localStorage cannot be used in a service worker.
  - The service worker can receive push messages from a server when the app is not active. This lets your app show push notifications to the user, even when it is not open in the browser.
  - The service worker can't access the DOM directly. To communicate with the page, the service worker uses the postMessage() method to send data and a "message" event listener to receive data.

Things to note about Service Worker:

  - A service worker is a programmable network proxy that lets you control how network requests from your page are handled.
  - Service workers only run over HTTPS. Because service workers can intercept network requests and modify responses, "man-in-the-middle" attacks could be very bad.
  - The service worker becomes idle when not in use and restarts when it's next needed. You cannot rely on a global state persisting between events. If there is information that you need to persist and reuse across restarts, you can use IndexedDB databases.
  - Service workers make extensive use of promises.



Service workers also depend on two APIs to work effectively: Fetch (a standard way to retrieve content from the network) and Cache (a persistent content storage for application data. This cache is persistent and independent from the browser cache or network status).

Service Workers limitations:

  - Only available on HTTPS sites (except localhost) to avoid MitM attacks.

Types of caches:
  - Pre-cache assets during installation (core of application shell architecture).
  - Provide a fallback for offline access.

Service workers acts as the base for advanced features like:
  - Channel messaging API: Allow web workers and service workers to communicate with each other.
  - Notifications API.
  - Push API.
  - Background sync.

Service worker lifecycle:

  - Registration:

```
if (!('serviceWorker' in navigator)) {
  console.log('Service Worker not supported');
  return;
}
navigator.serviceWorker.register('/service-worker.js')
.then(function(registration) {
  console.log('SW registered! Scope is:', registration.scope);
}); // .catch a registration error
```

  - Installation (install event)

```
self.addEventListener('install', function(event) {
  // Do stuff during install
});
```

  - Activation (activate event)

```
self.addEventListener('activate', function(event) {
  // Do stuff during activate
});
```

Once activated, the service worker will control all pages that load within its scope, and intercept corresponding network requests. 

However the pages in your app that are open will not be under the service worker’s scope since the service worker was not loaded when the pages opened. To put currently open pages under service worker control you must reload the page or pages. Until then, requests from this page will bypass the service worker and operate like they normally would. 

Service workers maintain control as long as there are pages open that are dependant on that specific version. This ensures that only one version of the service worker is running at any given time. 

Refreshing the page is not sufficient to transfer control to a new service worker, because there won’t be a time when the old service worker is not in use. 

The activation event is a good time to clean up stale data from existing caches for the application. 

Note: activation of a new service worker can be forced programatically with self.skipWaiting.


  - Fetch event

A fetch event is fired every time a resource is requested. In this example we listen for the fetch event, and instead of going to the network, return the requested resource from the cache (assuming it is there).

```
self.addEventListener('fetch', 
function(event) {
  event.respondWith(
    caches.match(event.request)
  );
});
```

### Service Workers Lab

See: ./lab/service-worker-lab/app

## Working with the Fetch API



## References

- Progressive Web Apps Training by Google: https://developers.google.com/web/ilt/pwa/
- The Offline Cookbook: https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/
- Promises, an Introduction: https://developers.google.com/web/fundamentals/primers/promises