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

Fetch is a modern replacement for XMLHttpRequest that lays the foundation for Progressive Web Apps.

It's based on promises (cleaner code).

Implements CORS (Cross-Origin Resource Sharing):
   
  - Browsers enforce the *Same-Origin Policy*:
    - Requests must match the page's scheme, hostname and port.
    - Exception: images, scripts, video/audio, embeds.
    - www.example.com requests JSON from www.json.com, fails unless the other origin gives permission.
  - Browser can request cross-origin access via CORS:
    - Adds origin header on request.
    - Server sends access-control-allow-origin if allowed.

Evaluating the success of responses is particularly important when using fetch because bad responses (like 404s) still resolve. The only time a fetch promise will reject is if the request was unable to complete. The previous code segment would only fall back to .catch if there was no network connection, but not if the response was bad (like a 404):

```
fetch('examples/example.json')
.then(function(response) {
  if (!response.ok) {
    throw Error(response.statusText);
  }
  // Do stuff with the response
})
.catch(function(error) {
  console.log('Looks like there was a problem: \n', error);
});
```

Now if the response object's ok property is false (indicating a non 200-299 response), the function throws an error containing response.statusText that triggers the .catch block. This prevents bad responses from propagating down the fetch chain.


Promise Chaining:

```
fetch('examples/example.json')
.then(function(response) {
  if (!response.ok) {
    throw Error(response.statusText);
  }
  // Read the response as json.
  return response.json();
})
.then(function(responseAsJson) {
  // Do stuff with the JSON
  console.log(responseAsJson);
})
.catch(function(error) {
  console.log('Looks like there was a problem: \n', error);
});
```

This code will be cleaner and easier to understand if it's abstracted into functions:

```
function logResult(result) {
  console.log(result);
}

function logError(error) {
  console.log('Looks like there was a problem: \n', error);
}

function validateResponse(response) {
  if (!response.ok) {
    throw Error(response.statusText);
  }
  return response;
}

function readResponseAsJSON(response) {
  return response.json();
}

function fetchJSON(pathToResource) {
  fetch(pathToResource) // 1
  .then(validateResponse) // 2
  .then(readResponseAsJSON) // 3
  .then(logResult) // 4
  .catch(logError);
}

fetchJSON('examples/example.json');
```

### Lab of Fetch API

See code at: labs/fetch-api-lab/app


## Caching Files with Service Worker

Caching provides a mechanism for storing request/response object pairs in the browser.

The Service Worker API comes with a Cache interface, that lets you create stores of responses keyed by request. While this interface was intended for service workers it is actually exposed on the window, and can be accessed from anywhere in your scripts. The entry point is caches.

All updates to items in the cache must be explicitly requested; items will not expire and must be deleted.

If the amount of cached data exceeds the browser's storage limit, the browser will begin evicting all data associated with an origin, one origin at a time, until the storage amount goes under the limit again.

### Common patterns of caching resources

#### On install (caching the application shell)

```
self.addEventListener('install', function(event) {
  event.waitUntil(
    caches.open(cacheName).then(function(cache) {
      return cache.addAll(
        [
          '/css/bootstrap.css',
          '/css/main.css',
          '/js/bootstrap.min.js',
          '/js/jquery.min.js',
          '/offline.html'
        ]
      );
    })
  );
});
```

Note: It is important to note that while this event is happening, any previous version of your service worker is still running and serving pages, so the things you do here must not disrupt that. For instance, this is not a good place to delete old caches, because the previous service worker may still be using them at this point.

event.waitUntil extends the lifetime of the install event until the passed promise resolves successfully. If the promise rejects, the installation is considered a failure and this service worker is abandoned (if an older version is running, it stays active).

#### On user interaction

If the whole site can't be taken offline, you can let the user select the content they want available offline (for example, a video, article, or photo gallery). One method is to give the user a "Read later" or "Save for offline" button.

```
document.querySelector('.cache-article').addEventListener('click', function(event) {
  event.preventDefault();
  var id = this.dataset.articleId;
  caches.open('mysite-article-' + id).then(function(cache) {
    fetch('/get-article-urls?id=' + id).then(function(response) {
      // /get-article-urls returns a JSON-encoded array of
      // resource URLs that a given article depends on
      return response.json();
    }).then(function(urls) {
      cache.addAll(urls);
    });
  });
});
```

The Cache API is available on the window object, meaning you don't need to involve the service worker to add things to the cache.

#### On network response

If a request doesn't match anything in the cache, get it from the network, send it to the page and add it to the cache at the same time.

```
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.open('mysite-dynamic').then(function(cache) {
      return cache.match(event.request).then(function (response) {
        return response || fetch(event.request).then(function(response) {
          cache.put(event.request, response.clone());
          return response;
        });
      });
    })
  );
});
```

Note: To allow for efficient memory usage, you can only read a response/request's body once. In the code above, .clone() is used to create a copy of the response that can be read separately. See What happens when you read a response? for more information.

### Serving files from the cache

#### Cache only

This approach is good for any static assets that are part of your app's main code (part of that "version" of your app). You should have cached these in the install event, so you can depend on them being there.

You should have cached these in the install event, so you can depend on them being there.

You don't often need to handle this case specifically. Cache falling back to network is more often the appropriate approach.

```
self.addEventListener('fetch', function(event) {
  event.respondWith(caches.match(event.request));
});
```

If a match isn't found in the cache, the response will look like a connection error.

#### Network only

This is the correct approach for things that can't be performed offline, such as analytics pings and non-GET requests. 

You don't often need to handle this case specifically. Cache falling back to network is more often the appropriate approach.

```
self.addEventListener('fetch', function(event) {
  event.respondWith(fetch(event.request));
});
```

#### Cache falling back to the network

If you're making your app offline-first, this is how you'll handle the majority of requests. Other patterns will be exceptions based on the incoming request.

```
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request).then(function(response) {
      return response || fetch(event.request);
    })
  );
});
```

This gives you the "Cache only" behavior for things in the cache and the "Network only" behaviour for anything not cached (which includes all non-GET requests, as they cannot be cached).

#### Network falling back to the cache

This is a good approach for resources that update frequently, and are not part of the "version" of the site (for example, articles, avatars, social media timelines, game leader boards). Handling network requests this way means the online users get the most up-to-date content, and offline users get an older cached version.

However, this method has flaws. If the user has an intermittent or slow connection they'll have to wait for the network to fail before they get content from the cache. This can take an extremely long time and is a frustrating user experience. See the next approach, Cache then network, for a better solution.

```
self.addEventListener('fetch', function(event) {
  event.respondWith(
    fetch(event.request).catch(function() {
      return caches.match(event.request);
    })
  );
});
```

#### Cache then network

This is also a good approach for resources that update frequently. This approach will get content on screen as fast as possible, but still display up-to-date content once it arrives.

This requires the page to make two requests: one to the cache, and one to the network. The idea is to show the cached data first, then update the page when/if the network data arrives.

```
var networkDataReceived = false;

startSpinner();

// fetch fresh data
var networkUpdate = fetch('/data.json').then(function(response) {
  return response.json();
}).then(function(data) {
  networkDataReceived = true;
  updatePage(data);
});

// fetch cached data
caches.match('/data.json').then(function(response) {
  if (!response) throw Error("No data");
  return response.json();
}).then(function(data) {
  // don't overwrite newer network data
  if (!networkDataReceived) {
    updatePage(data);
  }
}).catch(function() {
  // we didn't get cached data, the network is our last hope:
  return networkUpdate;
}).catch(showErrorMessage).then(stopSpinner());
```

Here is the code in the service worker:

```
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.open('mysite-dynamic').then(function(cache) {
      return fetch(event.request).then(function(response) {
        cache.put(event.request, response.clone());
        return response;
      });
    })
  );
});
```

#### Generic fallback

If you fail to serve something from the cache and/or network you may want to provide a generic fallback. This technique is ideal for secondary imagery such as avatars, failed POST requests, "Unavailable while offline" page.

```
self.addEventListener('fetch', function(event) {
  event.respondWith(
    // Try the cache
    caches.match(event.request).then(function(response) {
      // Fall back to network
      return response || fetch(event.request);
    }).catch(function() {
      // If both fail, show a generic fallback:
      return caches.match('/offline.html');
      // However, in reality you'd have many different
      // fallbacks, depending on URL & headers.
      // Eg, a fallback silhouette image for avatars.
    })
  );
});
```

### Removing outdated caches

Once a new service worker has installed and a previous version isn't being used, the new one activates, and you get an activate event. Because the old version is out of the way, it's a good time to delete unused caches.

```
self.addEventListener('activate', function(event) {
  event.waitUntil(
    caches.keys().then(function(cacheNames) {
      return Promise.all(
        cacheNames.filter(function(cacheName) {
          // Return true if you want to remove this cache,
          // but remember that caches are shared across
          // the whole origin
        }).map(function(cacheName) {
          return caches.delete(cacheName);
        })
      );
    })
  );
});

```

During activation, other events such as fetch are put into a queue, so a long activation could potentially block page loads. Keep your activation as lean as possible, only using it for things you couldn't do while the old version was active.

### Lab for Caching files

See labs/cache-api-lab/app


## Lighthouse PWA Analysis Tool

Lighthouse is an open-source analysis tool that provides insights and feedback for Progressive Web Apps.

As a Chrome extension or in the CLI:

```
npm install -g lighthouse
lighthouse https://pwa.rocks
```

### Lab Lighthouse

See labs/lighthouse-lab/app





## References

- Progressive Web Apps Training by Google: https://developers.google.com/web/ilt/pwa/
- The Offline Cookbook: https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/
- Promises, an Introduction: https://developers.google.com/web/fundamentals/primers/promises
- PWA rocks: http://pwa.rocks