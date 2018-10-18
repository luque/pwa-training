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


## Working with Promises

Promises offer a better way to handle asynchronous code in JavaScript.

The promise libraries listed above and promises that are part of the ES2015 JavaScript specification (also referred to as ES6) are all Promises/A+ compatible (See https://github.com/promises-aplus/promises-spec).

### Why use promises?

Asynchronous APIs are common in JavaScript to access the network or disk, to communicate with web workers and service workers, and even when using a timer. Most of these APIs use callback functions or events to communicate when a request is ready or has failed. While these techniques worked well in the days of simple web pages, they don't scale well to complete web applications.

#### Old way: using events

Using events to report asynchronous results has some major drawbacks:

  - It fragments your code into many pieces scattered among event handlers.
  - It's possible to get into race conditions between defining the handlers and receiving the events.
  - It often requires creating a class or using globals just to maintain state.

These make error handling difficult. For an example, look at any XMLHttpRequest code.

#### Old way: using callbacks

Another solution is to use callbacks, typically with anonymous functions. An example might look like the following:

```
function isUserTooYoung(id, callback) {
  openDatabase(function(db) {
    getCollection(db, 'users', function(col) {
      find(col, {'id': id}, function(result) {
        result.filter(function(user) {
          callback(user.age < cutoffAge);
        });
      });
    });
  });
}

```

The callback approach has two problems:

 - The more callbacks that you use in a callback chain, the harder it is to read and analyze its behavior.
 - Error handling becomes problematic. For example, what happens if a function receives an illegal value and/or throws an exception?


### Using promises

Promises provide a standardized way to manage asynchronous operations and handle errors. The above example becomes much simpler using promises:

```
function isUserTooYoung(id) {
  return openDatabase() // returns a promise
  .then(function(db) {return getCollection(db, 'users');})
  .then(function(col) {return find(col, {'id': id});})
  .then(function(user) {return user.age < cutoffAge;});
}
```

Think of a promise as an object that waits for an asynchronous action to finish, then calls a second function.

You can schedule that second function by calling .then() and passing in the function. When the asynchronous function finishes, it gives its result to the promise and the promise gives that to the next function (as a parameter).

Notice that there are several calls to .then() in a row. Each call to .then() waits for the previous promise, runs the next function, then converts the result to a promise if needed. This lets you painlessly chain synchronous and asynchronous calls. It simplifies your code so much that most new web specifications return promises from their asynchronous methods.

In the following example, we convert the asynchronous task of setting an image src attribute into a promise.

```
function loadImage(url) {
  // wrap image loading in a promise
  return new Promise(function(resolve, reject) {
    // A new promise is "pending"
    var image = new Image();
    image.src = url;
    image.onload = function() {
      // Resolving a promise changes its state to "fulfilled"
      // unless you resolve it with a rejected promise
      resolve(image);
    };
    image.onerror = function() {
      // Rejecting a promise changes its state to "rejected"
      reject(new Error('Could not load image at ' + url));
    };
  });
}
```

Here's a typical pattern for creating a promise:

```
var promise = new Promise(function(resolve, reject) {
  // do a thing, possibly async, then...

  if (/* everything turned out fine */) {
    resolve("Stuff worked!");
  }
  else {
    reject(Error("It broke"));
  }
});

```

The promise constructor takes one argument—a callback with two parameters: resolve and reject. Do something within the callback, perhaps async, then call resolve if everything worked, or otherwise call reject.

Like throw in plain old JavaScript, it's customary, but not required, to reject with an Error object. The benefit of Error objects is that they capture a stack trace, making debugging tools more helpful.

Here's one way to use that promise:

```
promise.then(function(result) {
  console.log("Success!", result); // "Stuff worked!"
}, function(err) {
  console.log("Failed!", err); // Error: "It broke"
});

```

The then() method takes two arguments, a callback for a success case, and another for the failure case. Both are optional, so you can add a callback for the success or failure case only.

A more common practice is to use .then() for success cases and .catch() for errors.

```
promise.then(function(result) {
  console.log("Success!", result);
}).catch(function(error) {
  console.log("Failed!", error);
})
```

There's nothing special about catch(), it's equivalent to then(undefined, func), but it's more readable. Note that the two code examples above do not behave the same way. The latter example is equivalent to:

There's nothing special about catch(), it's equivalent to then(undefined, func), but it's more readable. Note that the two code examples above do not behave the same way. The latter example is equivalent to:

```
promise.then(function(response) {
  console.log("Success!", response);
}).then(undefined, function(error) {
  console.log("Failed!", error);
})
```

The difference is subtle, but extremely useful. Promise rejections skip forward to the next then() with a rejection callback (or catch(), since they're equivalent). With then(func1, func2), func1 or func2 will be called, never both. But with then(func1).catch(func2), both will be called if func1 rejects, as they're separate steps in the chain.

### Promise chains

In a promise chain, the output of one function serves as input for the next.

The then() method schedules a function to be called when the previous promise is fulfilled. When the promise is fulfilled, .then() extracts the promise's value (the value the promise resolves to), executes the callback function, and wraps the returned value in a new promise.

When a promise rejects (or throws an exception), it jumps to the first .catch() call following the error and passes control to its function.

```
function processImage(imageName, domNode) {
  // returns an image for the next step. The function called in
  // the return statement must also return the image.
  // The same is true in each step below.
  return loadImage(imageName)
  .then(function(image) {
    // returns an image for the next step.
    return scaleToFit(150, 225, image);
  })
  .then(function(image) {
    // returns the image for the next step.
    return watermark('Google Chrome', image);
  })
  .then(function(image) {
    // Attach the image to the DOM after all processing has been completed.
    // This step does not need to return in the function or here in the
    // .then() because we are not passing anything on
    showImage(image);
  })
  .catch(function(error) {
    console.log('We had a problem in running processImage', error);
  });
}
```

You can use multiple catches in a promise chain to "recover" from errors in a promise chain. For example, the following code continues on with a fallback image if processImage or scaleToFit rejects:

```
function processImage(imageName, domNode) {
  return loadImage(imageName)
  .then(function(image) {
    return scaleToFit(150, 225, image);
  })
  .catch(function(error) {
    console.log('Error in loadImage() or scaleToFit()', error);
    console.log('Using fallback image');
    return fallbackImage();
  })
  .then(function(image) {
    return watermark('Google Chrome', image);
  })
  .then(function(image) {
    showImage(image);
  })
  .catch(function(error) {
    console.log('We had a problem with watermark() or showImage()', error);
  });
}
```

Not all promise-related functions have to return a promise. If the functions in a promise chain are synchronous, they don't need to return a promise.

The scaleToFit function is part of the image processing chain and doesn't return a promise:

```
function scaleToFit(width, height, image) {
  image.width = width;
  image.height = height;
  console.log('Scaling image to ' + width + ' x ' + height);
  return image;
}
```

### Promise.all

Often we want to take action only after a collection of asynchronous operations have completed successfully. Promise.all returns a promise that resolves if all of the promises passed into it resolve. If any of the passed-in promises reject, then Promise.all rejects with the reason of the first promise that rejected. This is very useful for ensuring that a group of asynchronous actions complete before proceeding to another step.

```
var promise1 = getJSON('/users.json');
var promise2 = getJSON('/articles.json');

Promise.all([promise1, promise2]) // Array of promises to complete
.then(function(results) {
  console.log('all data has loaded');
})
.catch(function(error) {
  console.log('one or more requests have failed: ' + error);
});
```

### Promise.race

Another promise method that you may see referenced is Promise.race. Promise.race takes a list of promises and settles as soon as the first promise in the list settles. If the first promise resolves, Promise.race resolves with the corresponding value, if the first promise rejects, Promise.race rejects with the corresponding reason. The following code shows example usage of Promise.race:

```
Promise.race([promise1, promise2])
.then(function(value) {
  console.log(value);
})
.catch(function(reason) {
  console.log(reason);
});
```

If one of the promises resolves first, the then block executes and logs the value of the resolved promise. If one of the promises rejects first, the catch block executes and logs the reason for the promise rejection.


## Working with IndexedDB

We are using Jake Archibald's IndexedDB Promised library, which is very similar to the IndexedDB API, but uses promises rather than events. More info at: https://github.com/jakearchibald/idb

### What is IndexedDB from MDN:

"IndexedDB is a low-level API for client-side storage of significant amounts of structured data, including files/blobs. This API uses indexes to enable high performance searches of this data. While DOM Storage is useful for storing smaller amounts of data, it is less useful for storing larger amounts of structured data. IndexedDB provides a solution."

Each IndexedDB database is unique to an origin (typically, this is the site domain or subdomain), meaning it cannot access or be accessed by any other origin.

Data storage limits are usually quite large, if they exist at all, but different browsers handle limits and data eviction differently. See https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Browser_storage_limits_and_eviction_criteria for more information.

For example, in Firefox the maximum browser storage space is dynamic --it is based on your hard drive size. The global limit is calculated as 50% of free disk space. So if your hard drive is 500 GB, then the total storage for a browser is 250 GB. If this is exceeded, a process called origin eviction comes into play, deleting an entire origin's worth of data until the storage amount goes under the limit again. There's also another limit called group limit — this is defined as 20% of the global limit, but it has a minimum of 10 MB and a maximum of 2 GB. Each origin is part of a group (group of origins). There's one group for each eTLD+1 domain.

### IndexedDB terms

Database - This is the highest level of IndexedDB. It contains the object stores, which in turn contain the data you would like to persist. You can create multiple databases with whatever names you choose, but generally there is one database per app.

Object store - An object store is an individual bucket to store data. You can think of object stores as being similar to tables in traditional relational databases. Typically, there is one object store for each 'type' (not JavaScript data type) of data you are storing. For example, given an app that persists blog posts and user profiles, you could imagine two object stores. Unlike tables in traditional databases, the actual JavaScript data types of data within the store do not need to be consistent (for example, if there are three people in the 'people' object store, their age properties could be 53, 'twenty-five', and unknown ).

Index - An Index is a kind of object store for organizing data in another object store (called the reference object store) by an individual property of the data. The index is used to retrieve records in the object store by this property. For example, if you're storing people, you may want to fetch them later by their name, age, or favorite animal.

Transaction - A transaction is wrapper around an operation, or group of operations, that ensures database integrity. If one of the actions within a transaction fail, none of them are applied and the database returns to the state it was in before the transaction began. All read or write operations in IndexedDB must be part of a transaction. This allows for atomic read-modify-write operations without worrying about other threads acting on the database at the same time.

Cursor - A mechanism for iterating over multiple records in database.


### Checking for IndexedDB support

Because IndexedDB isn't supported by all browsers, we need to check that the user's browser supports it before using it. The easiest way is to check the window object:

```
if (!('indexedDB' in window)) {
  console.log('This browser doesn\'t support IndexedDB');
  return;
}
```

### Opening a database

```
idb.open(name, version, upgradeCallback)
```

This method returns a promise that resolves to a database object. When using idb.open, you provide a name, version number, and an optional callback to set up the database.

An example:

```
(function() {
  'use strict';

  //check for support
  if (!('indexedDB' in window)) {
    console.log('This browser doesn\'t support IndexedDB');
    return;
  }

  var dbPromise = idb.open('test-db1', 1);

})();
```

### Working with object stores

To ensure database integrity, object stores can only be created and removed in the callback function in idb.open.

The callback receives an instance of UpgradeDB, a special object in the IDB Promised library that is used to create object stores.

```
upgradeDb.createObjectStore('storeName', options);
```

Below is an example of the createObjectStore method:

```
(function() {
  'use strict';

  //check for support
  if (!('indexedDB' in window)) {
    console.log('This browser doesn\'t support IndexedDB');
    return;
  }

  var dbPromise = idb.open('test-db2', 1, function(upgradeDb) {
    console.log('making a new object store');
    if (!upgradeDb.objectStoreNames.contains('firstOS')) {
      upgradeDb.createObjectStore('firstOS');
    }
  });

})();
```

When you define object stores, you can define how data is uniquely identified in the store using the primary key. You can define a primary key by either defining a key path, or by using a key generator.

A key path is a property that always exists and contains a unique value:

```
upgradeDb.createObjectStore('people', {keyPath: 'email'});
```

You could also use a key generator, such as autoIncrement. The key generator creates a unique value for every object added to the object store. By default, if we don't specify a key, IndexedDB creates a key and stores it separately from the data.

```
upgradeDb.createObjectStore('notes', {autoIncrement:true});
```

This example creates an object store called "notes" and sets the primary key to be assigned automatically as an auto incrementing number.

```
upgradeDb.createObjectStore('logs', {keyPath: 'id', autoIncrement:true});
```

```
function() {
  'use strict';

  //check for support
  if (!('indexedDB' in window)) {
    console.log('This browser doesn\'t support IndexedDB');
    return;
  }

  var dbPromise = idb.open('test-db3', 1, function(upgradeDb) {
    if (!upgradeDb.objectStoreNames.contains('people')) {
      upgradeDb.createObjectStore('people', {keyPath: 'email'});
    }
    if (!upgradeDb.objectStoreNames.contains('notes')) {
      upgradeDb.createObjectStore('notes', {autoIncrement: true});
    }
    if (!upgradeDb.objectStoreNames.contains('logs')) {
      upgradeDb.createObjectStore('logs', {keyPath: 'id', autoIncrement: true});
    }
  });
})();
```

### Defining indexes

Indexes are a kind of object store used to retrieve data from the reference object store by a specified property.

An index lives inside the reference object store and contains the same data, but uses the specified property as its key path instead of the reference store's primary key.

To create an index, call the createIndex method on an object store instance:

```
objectStore.createIndex('indexName', 'property', options);
```

The final argument lets you define two options that determine how the index operates: unique and multiEntry.  If unique is set to true, the index does not allow duplicate values for a single key. multiEntry determines how createIndex behaves when the indexed property is an array. If it's set to true, createIndex adds an entry in the index for each array element. Otherwise, it adds a single entry containing the array.

```
(function() {
  'use strict';

  //check for support
  if (!('indexedDB' in window)) {
    console.log('This browser doesn\'t support IndexedDB');
    return;
  }

  var dbPromise = idb.open('test-db4', 1, function(upgradeDb) {
    if (!upgradeDb.objectStoreNames.contains('people')) {
      var peopleOS = upgradeDb.createObjectStore('people', {keyPath: 'email'});
      peopleOS.createIndex('gender', 'gender', {unique: false});
      peopleOS.createIndex('ssn', 'ssn', {unique: true});
    }
    if (!upgradeDb.objectStoreNames.contains('notes')) {
      var notesOS = upgradeDb.createObjectStore('notes', {autoIncrement: true});
      notesOS.createIndex('title', 'title', {unique: false});
    }
    if (!upgradeDb.objectStoreNames.contains('logs')) {
      var logsOS = upgradeDb.createObjectStore('logs', {keyPath: 'id',
        autoIncrement: true});
    }
  });
})();
```

### Working with data

All data operations in IndexedDB are carried out inside a transaction. Each operation has this form:

   - Get database object
   - Open transaction on database
   - Open object store on transaction
   - Perform operation on object store

Transactions are specific to one or more object stores, which we define when we open the transaction.

They can be read-only or read and write.

#### Creating data

To create data, call the add method on the object store and pass in the data you want to add.

```
someObjectStore.add(data, optionalKey);
```

The data parameter can be data of any type: a string, number, object, array, and so forth. The only restriction is if the object store has a defined keypath, the data must contain this property and the value must be unique.

The add method returns a promise that resolves once the object has been added to the store.

Add occurs within a transaction, so even if the promise resolves successfully it doesn't necessarily mean the operation worked. Remember, if one of the actions in the transaction fails, all of the operations in the transaction are rolled back. To be sure that the add operation was carried out, we need to check if the whole transaction has completed using the transaction.complete method. transaction.complete is a promise that resolves when the transaction completes and rejects if the transaction errors.

```
dbPromise.then(function(db) {
  var tx = db.transaction('store', 'readwrite');
  var store = tx.objectStore('store');
  var item = {
    name: 'sandwich',
    price: 4.99,
    description: 'A very tasty sandwich',
    created: new Date().getTime()
  };
  store.add(item);
  return tx.complete;
}).then(function() {
  console.log('added item to the store os!');
});
```

#### Reading data

To read data, call the get method on the object store. The get method takes the primary key of the object you want to retrieve from the store:

```
someObjectStore.get(primaryKey);
```

As with add, the get method returns a promise and must happen within a transaction:

```
dbPromise.then(function(db) {
  var tx = db.transaction('store', 'readonly');
  var store = tx.objectStore('store');
  return store.get('sandwich');
}).then(function(val) {
  console.dir(val);
});
```

#### Updating data

```
dbPromise.then(function(db) {
  var tx = db.transaction('store', 'readwrite');
  var store = tx.objectStore('store');
  var item = {
    name: 'sandwich',
    price: 99.99,
    description: 'A very tasty, but quite expensive, sandwich',
    created: new Date().getTime()
  };
  store.put(item);
  return tx.complete;
}).then(function() {
  console.log('item updated!');
});
```

#### Deleting data

```
dbPromise.then(function(db) {
  var tx = db.transaction('store', 'readwrite');
  var store = tx.objectStore('store');
  store.delete(key);
  return tx.complete;
}).then(function() {
  console.log('Item deleted');
});
```

### Getting all the data

So far we have only retrieved objects from the store one at a time. We can also retrieve all of the data (or subset) from an object store or index using either the getAll method or using cursors.

#### Using getAll method

The simplest way to retrieve all of the data is to call the getAll method on the object store or index, like this:

```
someObjectStore.getAll(optionalConstraint);
```

As with all other database operations, this operation happens inside a transaction.

```
dbPromise.then(function(db) {
  var tx = db.transaction('store', 'readonly');
  var store = tx.objectStore('store');
  return store.getAll();
}).then(function(items) {
  console.log('Items by name:', items);
});
```

#### Using cursors

A cursor selects each object in an object store or index one by one, letting you do something with the data as it is selected.

Cursors, like the other database operations, work within transactions.

We create the cursor by calling the openCursor method on the object store, like this:

```
someObjectStore.openCursor(optionalKeyRange, optionalDirection);
```

This method returns a promise that resolves with a cursor object representing the first object in the object store or undefined if there is no object.

To move on to the next object in the object store, we call cursor.continue. This returns a promise that resolves with the next object, or undefined if there are no more objects. We put this inside a loop to move through all of the entries in the store one by one.

The optional key range in the openCursor method limits the iteration to a subset of the objects in the store.

The direction option can be next or prev, specifying forward or backward traversal through the data.

```
dbPromise.then(function(db) {
  var tx = db.transaction('store', 'readonly');
  var store = tx.objectStore('store');
  return store.openCursor();
}).then(function logItems(cursor) {
  if (!cursor) {
    return;
  }
  console.log('Cursored at:', cursor.key);
  for (var field in cursor.value) {
    console.log(cursor.value[field]);
  }
  return cursor.continue().then(logItems);
}).then(function() {
  console.log('Done cursoring');
});
```

### Working with ranges and indexes

Indexes let us fetch the data in an object store by a property other than the primary key.

We can create an index on any property (which becomes the keypath for the index), specify a range on that property, and get the data within the range using the getAll method or a cursor.

We define the range using the IDBKeyRange object:

```
IDBKeyRange.lowerBound(indexKey);

or

IDBKeyRange.upperBound(indexKey);

or

IDBKeyRange.bound(lowerIndexKey, upperIndexKey);
```

```
function searchItems(lower, upper) {
  if (lower === '' && upper === '') {return;}

  var range;
  if (lower !== '' && upper !== '') {
    range = IDBKeyRange.bound(lower, upper);
  } else if (lower === '') {
    range = IDBKeyRange.upperBound(upper);
  } else {
    range = IDBKeyRange.lowerBound(lower);
  }

  dbPromise.then(function(db) {
    var tx = db.transaction(['store'], 'readonly');
    var store = tx.objectStore('store');
    var index = store.index('price');
    return index.openCursor(range);
  }).then(function showRange(cursor) {
    if (!cursor) {return;}
    console.log('Cursored at:', cursor.key);
    for (var field in cursor.value) {
      console.log(cursor.value[field]);
    }
    return cursor.continue().then(showRange);
  }).then(function() {
    console.log('Done cursoring');
  });
}
```

### Using database versioning

When we call idb.open, we can specify the database version number in the second parameter. If this version number is greater than the version of the existing database, the upgrade callback executes, allowing us to add object stores and indexes to the database.

The UpgradeDB object has a special oldVersion property, which indicates the version number of the database existing in the browser.

```
var dbPromise = idb.open('test-db7', 2, function(upgradeDb) {
  switch (upgradeDb.oldVersion) {
    case 0:
      upgradeDb.createObjectStore('store', {keyPath: 'name'});
    case 1:
      var peopleStore = upgradeDb.transaction.objectStore('store');
      peopleStore.createIndex('price', 'price');
  }
});
```

### localStorage vs IndexedDB

On the surface the two technologies may seem directly comparable, however if you spend some time with them you'll soon realize they are not. They were designed to achieve a similar goal, client side storage, but they approach the task at hand from significantly different perspectives and work best with different amounts of data.

localStorage, or more accurately Web Storage, was designed for smaller amounts of data. It's essentially a strings only key - value storage, with a simplistic synchronous API. That last part is key. Although there's nothing in the specification that prohibits an asynchronous DOM Storage, currently all implementations are synchronous (i.e. blocking requests). Even if you didn't mind using a naive key - value storage for larger amounts of data, your clients will mind waiting forever for your application to load.

indexedDB, on the other hand, was designed to work with significantly larger amounts of data. First, in theory, it provides both a synchronous and an asynchronous API. In practice, however, all current implementations are asynchronous, and requests will not block the user interface from loading. Additionally, indexedDB, as the name reveals, provides indexes. You can run rudimentary queries on your database and fetch records by looking up theirs keys in specific key ranges. indexedDB also supports transactions, and provides simple types (e.g. Date).

At the end of the day, it's completely up to you if you use DOM Storage or indexedDB, or both, in your application. A good use case for DOM Storage would be to store simple session data, for example a user's name, and save you some requests to your actual database. indexedDB's additional features, on the other hand, could help you store all the data you need for your application to work offline. 


## Live Data in the Service Worker

A general guideline for data storage is that URL addressable resources should be stored with the Cache interface, and other data should be stored with IndexedDB.

For example HTML, CSS, and JS files should be stored in the cache, while JSON data should be stored in IndexedDB.

### Why Cache API and IndexedDB?

Both are asynchronous and accessible in service workers, web workers, and the window interface.

IndexedDB is widely supported, and the Cache interface is supported in Chrome, Firefox, Opera, and Samsung Internet.

### How Much Can You Store?

Different browsers allow different amounts of offline storage.


The service worker activation event is a good time to create a database. Creating a database during the activation event means that it will only be created (or opened, if it already exists) when a new service worker takes over, rather than each time the app runs (which is inefficient). It's also likely better than using the service worker's installation event, since the old service worker will still be in control at that point, and there could be conflicts if a new database is mixed with an old service worker.

```
function createDB() {
  idb.open('products', 1, function(upgradeDB) {
    var store = upgradeDB.createObjectStore('beverages', {
      keyPath: 'id'
    });
    store.put({id: 123, name: 'coke', price: 10.99, quantity: 200});
    store.put({id: 321, name: 'pepsi', price: 8.99, quantity: 100});
    store.put({id: 222, name: 'water', price: 11.99, quantity: 300});
  });
}

self.addEventListener('activate', function(event) {
  event.waitUntil(
    createDB()
  );
});
```

The service worker installation event is a good time to cache static assets like these. This ensures that all the resources a service worker is expected to have are cached when the service worker is installed.

```
function cacheAssets() {
  return caches.open('cache-v1')
  .then(function(cache) {
    return cache.addAll([
      '.',
      'index.html',
      'styles/main.css',
      'js/offline.js',
      'img/coke.jpg'
    ]);
  });
}

self.addEventListener('install', function(event) {
  event.waitUntil(
    cacheAssets()
  );
});

self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request).then(function(response) {
      // Check cache but fall back to network
      return response || fetch(event.request);
    })
  );
});

```

## Using Workbox

Workbox is the successor to *sw-precache* and *sw-toolbox*. It is a collection of libraries and tools used for *generating a service worker*, *precaching*, *routing*, and *runtime-caching*.

Workbox also includes modules for easily integrating *background sync* and *Google Analytics* into your service worker.

### Why?

Workbox is a library that bakes in a set of best practices and removes the boilerplate every developer writes when working with service workers:
    - Precaching
    - Runtime caching
    - Strategies
    - Request routing
    - Background sync
    - Helpful debugging
    - Greater flexibility and feature set than sw-precache and sw-toolbox

### Who is using Workbox?

Pinterest's PWA:
  - Precaching.
  - Fast navigations (some navigation requests bypass the network).

WIRED's PWA:
  - workbox-cli generates their service worker each time they build.
  - offline reading experience: offline article reading list from window.caches

### Next 3.0 release features

  - Improved background sync
  - Smaller runtime payloads: workbox-sw 2.0 47KB -> workbox-sw 3.0 15KB


### Lab

Use the workbox-sw.js library and the workbox-build Node.js module to build an offline-capable PWA.

See labs/workbox-lab/project

  - Write a basic service worker using workbox-sw

  Rather than adding files to the list manually, workbox-build can generate the manifest for us. Using a tool like workbox-build has multiple advantages:

  1. The tool can be integrated into our build process. Adding workbox-build to our build process eliminates the need for manual updates to the precache manifest each time that we update the apps files.
  2. workbox-build automatically adds "revision hashes" to the files in the manifest entries. The revision hashes enable Workbox to intelligently track when files have been modified or are outdated, and automatically keep caches up to date with the latest file versions. Workbox can also remove cached files that are no longer in the manifest, keeping the amount of data stored on a user's device to a minimum. You'll see what workbox-build and the file revision hashes look like in the next section.

While we are using workbox-build with gulp in this lab, Workbox also supports tools like webpack with workbox-webpack-plugin and npm-build processed with workbox-cli.

  - Inject a manifest into the service worker

  Now we need to configure workbox-build to inject a precache manifest in the precacheAndRoute call in the service worker file.

  - Register and test the service worker

  In addition to precaching, the precacheAndRoute method sets up an implicit cache-first handler, ensuring that the precached resources are served offline.

  - Add routes to the service worker

  workbox-sw.js has a routing module that lets you easily add routes to your service worker.

```
workbox.routing.registerRoute(
  /(.*)articles(.*)\.(?:png|gif|jpg)/,
  workbox.strategies.cacheFirst({
    cacheName: 'images-cache',
    plugins: [
      new workbox.expiration.Plugin({
        maxEntries: 50,
        maxAgeSeconds: 30 * 24 * 60 * 60, // 30 Days
      })
    ]
  })
);
```

  - Use a customized networkFirst cache strategy

  Sometimes content must always be kept up-to-date (e.g., news articles, stock figures, etc.). For this kind of data, the cacheFirst strategy is not the best solution. Instead, we can use the networkFirst strategy to fetch the newest content first, and only if that fails does the service worker get old content from the cache.

```
const articleHandler = workbox.strategies.networkFirst({
  cacheName: 'articles-cache',
  plugins: [
    new workbox.expiration.Plugin({
      maxEntries: 50,
    })
  ]
});

workbox.routing.registerRoute(/(.*)article(.*)\.html/, args => {
  return articleHandler.handle(args);
});
```

  - Handle invalid responses

  The handle method returns a promise resolving with the response, so we can access the response with a .then.

```
.then(response => {
    if (!response) {
      return caches.match('pages/offline.html');
    } else if (response.status === 404) {
      return caches.match('pages/404.html');
    }
    return response;
  });
```

  The .then statement receives the response passed in from the handle method. If the response doesn't exist, then it means the user is offline and the response was not previously cached. 


## Responsive Design

Users interact with the web using an increasingly diverse range of devices. These devices all have different viewport sizes, pixel widths and densities, scroll and zoom capabilities, etc.


"Responsive design", coined by Ethan Marcotte in A List Apart, refers to a set of technologies that make sites adapt to the form factor the site is viewed in. These technologies are:
 - Fluid Grids: All the content measurements are expressed in relative units to ensure that the layout will remain as consistent as possible across form factors.
 - Flexible images: All images are expressed using relative units so that they resize along with the content.
 - Media Queries: A way to rearrange your layout in CSS without modifying your HTML.

### Progressive Enhancement:

We need to make sure we provide a good default experience with a technology or language (such as HTML) that is widely supported, and then conditionally add the features we want based on support for those features.

Remember: The content is the most important part of our app. The goal of progressive enhancement is to preserve the content, so that if something breaks, our content will still be there in our default experience. 

### PWA golden rule:

Never make users scroll horizontally.

Sounds basic, but lots of site break this rule by making images, inputs and other large elements on the page with fixed sizes.

​Using relative measurement units (em, rem, or percentages) will reduce the severity of this issue.


```
img {
  max-width: 100%;
}
```

### Viewport meta tag:

```
<meta name="viewport" content="width=device-width, initial-scale=1">
```

Setting the viewport meta tag correctly, browsers scale down a page to fit a (virtual) 980px wide viewport either by zooming into a portion of the page that matches the available screen size or by shrinking the page until it fits into the available viewport, rendering the text impossible to read.

The meta viewport tag is designed to work with responsive layouts. If you use it in a fixed size layout it will break things until you convert the site to a responsive layout. 

### Relative sizing is not the (full) solution:

For a while developers thought that relative sizing layout and images was enough. This is not always the case… Sometimes the design is too big on desktop and too small on mobile. 

### Media queries

This is why media queries were invented.

It's a simple concept: modify the layout of your site based on different criteria (mostly width but not exclusively).

If we only had to worry about devices this may be all the queries that we need to worry about, right?


```
/* elements in common for phones and tablets */

@media (max-width: 480px) {
  /* layout for phones */
}

@media (max-width: 720px) { 
  /* layout for tablets */  
}
```

If we want to cover our bases with media queries then we need to worry about more than just phones and tablets. We have 4 queries to worry about for phone, tablet, laptop and widescreen devices.  What happens if the phone or tablet is placed in landscape mode instead of portrait?  What happens with retina devices?

  - 320,480px: phone
  - 768px: tablet
  - 922px: laptop
  - 1200px: widescreen

We will soon find ourselves with a large number of media queries and a nightmare of managing a large CSS file and how it changes.

### Match breakpoints to content

- Start small
- Add major breakpoints
- Add minor breakpoints if necessary
- Optimize for reading: 70–80 characters per line

### Content-based media queries

1. Understand your users
2. Understand your target devices
3. Design content
4. Use media queries to select layouts

That doesn't mean you stop thinking about devices and device classes: you might want one column for phones, two columns for tablets, three columns for desktop — or whatever.

### Mobile-first media queries

```
/* small */

@media (min-width: 480px) {
  /* medium */
}
@media (min-width: 720px) { 
  /* large */  
}
```

Remember the earlier media queries example? 

In the mobile-first world of PWAs, we need to turn that around. Here we are using min-width instead of max-width in our media queries to handle medium and large form factors as the exception.

calc():

```
img.thumb {
  margin-right: 10px;
  max-width: 400px;
  width: calc((100% - 10px) / 2);
}
img.thumb: last-of-type {
  margin-right: 0;
}
```

### Content first

Responsive design is about more than just changing layouts.

Media queries can do more than just change layouts, they also allow designers to manipulate content, depending on the viewport size and device type.

On a phone, you might want to make sure page content is visible when the user goes to your home page — so you might opt for a hamburger menu for navigation, and move ads lower on the page.


  - Reorder
  - Reposition
  - Replace
  - Remove (last resort): A better alternative to removing content is to use content appropriate to the displays you’re targeting

### Keep it simple

Help users get the content.

Every step to get to content loses 20% of users.

### CSS Flexbox

 - Flexible sizing and alignment.
 - Element reordering.
 - Better performance than floats.

```
.container {
  display: flex;
  flex-flow: row wrap;
}

.child1, .child2, .child3, .child4, .child5 {
  width: 100%;
}

@media (min-width: 600px) {
  .child2, .child3, .child4, .child5 {
    width: 50%;
  }
}

@media (min-width: 800px) {
  .container {
    margin-left: auto;
    margin-right: auto;
    width: 800px;
  }
  .child1 { width: 60%; }
  .child2 { width: 40%; }
  .child3, .child4, .child5 { width: 33.33%; }
}
```

- CSS Flexbox ordering:

```
.container {
  display: flex;
  flex-flow: row wrap;
}

.child1, .child2, .child3 {
  width: 100%;
}

@media (min-width: 600px) {
  .child1 {
    order: 2;
    width: 60%;
  }
  .child2 {
    order: 1;
    width: 40%;
  }
}
```

### CSS Grid

 - Works with CSS Flexbox
 - Optimized for UI design
 - Separation of layout and content


## Responsive images

There are two major issues to consider when working with images on the web: performance and responsiveness.

### What are responsive images?

A method for providing the browser with multiple image sources depending on display density, size of the image element in the page, or any number of other factors. 
-- Jason Grigsby

### General principles

 - Avoid images whenever possible.
 - Use vector formats where possible: SVG or icon fonts.
 - Use the lowest possible resolution and quality.
 - Use the right format for the image type: WebP, PNG, JPEG.

Incorporate lossless compression tools in your workflow.

### Elements and attributes

 - <picture>
 - <source>
 - srcset
 - sizes

### The <picture> and <source> elements

The picture and source elements enable us to provide alternative sources for the same resource. 

The browser will stop and load the first source element that it understands and load that image. If the browser can’t read the files specified in the source elements or if the browser doesn’t support the picture and source elements the default image will be loaded. 

```
<picture>
    <source srcset="kittens.webp" type="image/webp">
    <source srcset="kittens.jpg" type="image/jpeg">
    <img src="kittens.jpg" alt="Two grey tabby kittens">
</picture>
```

The source element can take an optional media attribute where you can specify a media query. When that media query is triggered then the image in the associated srcset attribute will be loaded.

```
<picture>
    <source media="(min-width: 650px)" srcset="kitten-lg.png">
    <source media="(min-width: 465px)" srcset="kitten-md.png">
    <img src="kitten-small.png" alt="Cute kitten">
</picture>
```

### The srcset attribute

In this example we use srcset in an img element to handle different pixel densities. 

If the display is a standard resolution display then we’ll use the 1x image, if the monitor is a 2x high DPI screen we’ll use. If the browser can’t understand the srcset attribute then it will use the regular src attribute and load our 1x image. 

Using the x descriptor, you’ll always get the same image on the devices with similar device-pixel ratio, regardless if it’s a 15 inch laptop or a cell phone which have the same device-pixel ratio.

```
<img src="wallaby_1x.jpg" alt="Wallaby"
    srcset="wallaby_1x.jpg 1x, wallaby_2x.jpg 2x" />
```

#### The width descriptor

For a browser, there's a Catch-22 when it comes to choosing which image to download: the browser needs to know the dimensions of each image, but it can't know that without downloading each image to check.

The w unit *tells* the browser the width of each image in pixels, thereby enabling the browser to choose the right image to retrieve – depending on the screen pixel density and the viewport size. 

```
<img src="small.jpg" alt="Wallaby"
    srcset="small.jpg 500w, med.jpg 1000w, 
    large.jpg 1500w">
```

### The sizes attribute

The sizes attribute tells the browser the size or sizes of the element the srcset is attached to so that the browser can use the appropriate image. In this case, we are telling the browser that the image will be displayed at 50% of the viewport width.

```
<img src="small.jpg" srcset="small.jpg 500w, medium.jpg 1000w, large.jpg 1500w" sizes="50vw" alt="Wallaby" />
```

### All together

We use all the techniques we discussed. Combining media queries and srcset to specify images for different viewports and also providing different images for different pixel densities. 

```
<picture>
  <source media="(min-width: 1000px)" srcset="bird_large_1x.jpg 1x, bird_large_2x.jpg 2x">
  <source media="(min-width: 500px)" srcset="bird_med_1x.jpg 1x, bird_med_2x.jpg 2x">
  <img src="bird_small.jpg" alt="A colorful bird">
</picture>
```

>> Tools like **responsivebreakpoints.com** will generate the images and the corresponding code for you so you don’t have to do 










## References

- Progressive Web Apps Training by Google: https://developers.google.com/web/ilt/pwa/
- The Offline Cookbook: https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/
- Promises, an Introduction: https://developers.google.com/web/fundamentals/primers/promises
- PWA rocks: http://pwa.rocks
- Offline Storage for PWAs: https://medium.com/dev-channel/offline-storage-for-progressive-web-apps-70d52695513c
- Workbox: https://developers.google.com/web/tools/workbox/
- Workbox: Flexible PWA Libraries: https://www.youtube.com/watch?time_continue=11&v=DtuJ55tmjps
- Responsive Web Design Patterns: https://developers.google.com/web/fundamentals/design-and-ux/responsive/patterns
- 7 Habits of Highly Effective Media Queries: http://bradfrost.com/blog/post/7-habits-of-highly-effective-media-queries/#content
- Responsive Breakpoints Tool: http://responsivebreakpoints.com
- Using WebP Images: https://css-tricks.com/using-webp-images/