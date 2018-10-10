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

ES2015:

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

Fetch API:

```
fetch('/examples/example.json')
.then(response => {
  return response.json();
}) // .then do something with the data
.catch(error => {
  console.log('Fetch failed', error);
});
```



## References

- Progressive Web Apps Training by Google: https://developers.google.com/web/ilt/pwa/