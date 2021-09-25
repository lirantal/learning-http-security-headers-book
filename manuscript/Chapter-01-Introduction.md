# Introduction

## Requirements

If you have a development environment set with Node.js, git, npm, and working Internet connectivity, you're all set to get started!

Some exercises require work with a valid HTTPS-enabled website, for which we defer to Heroku as the web hosting platform due to its ease of use and supporting simultaneously both HTTP and HTTPS web hosting.

### Web knowledge prerequisites

It is expected that you have basic knowledge of HTTP, such as the meaning of HTTP headers, an HTTP request, and response, and general knowledge of how the web works in terms of the interactions between a web server and a web client (the browser).

It is also expected that you have Chrome installed, but not mandatory. This book refers to Chrome's DevTools, and includes screenshots using the Chrome browser. If you are using any other browser and can make the parallels yourself, you should be fine moving forward with the book.

### A JavaScript and Node.js development environment

This book uses the Express web application framework for Node.js to create web applications and set headers using open source modules from the npm ecosystem.

It is expected that you have a working development environment with a supported Node.js version (LTS), along with the `npm` command-line utility.

You'll also need `git` to clone example repositories used along with the exercises if you wish to practice locally.

### Heroku hosting

While you are free to deploy the Node.js web application provided in the code references to any web hosting you'd like, such as [Vercel](https://vercel.com), or [Netlify](https://www.netlify.com) - the exercises explain how to use a free Heroku account to deploy.

## Headers as browser security controls

What are HTTP security headers? How can they be useful to secure web applications and specially used as low-hanging fruits, which are easy to implement? At the same time, they may break a web application if not applied correctly.

### Intro

Developing web applications means that our application depends on communication protocols that already have a set of standards defined and implemented for how to transfer data and how to manage it in a secure manner.

Browsers utilize headers sent over HTTP (secure HTTP connections mostly) to enforce and confirm such communication standards as well as security policies. Making use of these HTTP headers to increase security for the code running on the browser client-side is a quick and efficient method to mitigate security vulnerabilities and add defense in depth.

### Security headers and Node.js

In this book, we will introduce browser security controls by implementing HTTP headers for increased security.

We'll learn about Helmet as a library that can be easily added to any Express project and configure it to provide additional security for Node.js web applications.

The HTTP security headers that we will review are:

- Strict Transport Security
- X Frame Options
- Content Security Policy
- X XSS Protection
- X Content Type Options

### Security headers caveats

Utilizing security headers can be a great strategy to help prevent security vulnerabilities, but a common mistake is to rely solely on them to mitigate such issues. This is because responding to a request with a security header depends on the browser to actually support, implement, and adhere to the specification to enforce it. You may consult the [Strict Transport Security browser compatibility matrix](https://caniuse.com/#feat=stricttransportsecurity) to verify if the browsers used for your web application are supported.

As such, security headers should be used as a defense in depth security mechanism that helps in adding a security control, but they shouldn't be the actual security control to defend against vulnerabilities like Cross-site Scripting.

I> Defense in Depth
I>
I> A defense in depth is a security concept in which multiple layers of security controls are placed in order to create a better security posture.

## Helmet - a Node.js package to set HTTP security headers

HTTP security headers are a generic tool that can be employed by any technology at the HTTP medium, such as load balancers, an API gateway, reverse proxies, or web application frameworks.

[Helmet](https://helmetjs.github.io) is an open source project which comprises a collection of HTTP middleware functions that configure HTTP headers by setting the HTTP response object accordingly.

If you're building Node.js web applications with the help of [Express](http://expressjs.com), then Helmet is the go-to npm package to use and all source code examples in the book will follow its usage. If you're using other frameworks, such as Fastify, then consult the source-code example in the follow sub-sections.

### Helmet and Express

If you're using an Express web application setup, begin by installing the Helmet module:

```bash
npm install --save helmet
```

Then, continue to instantiate an Express application object, and set an application middleware using Helmet. Specifically, in this example, we're setting the X-Frame-Options using Helmet's built-in `frameguard` method:

```js
const express = require("express");
const helmet = require("helmet");

const app = express();

app.use(
  helmet.frameguard({
    action: "sameorigin",
  })
);
```

### Helmet and Fastify

If you're using the [Fastify](https://github.com/fastify/fastify) web application framework, begin by installing the Helmet wrapper module [fastify-helmet](https://github.com/fastify/fastify-helmet):

```bash
npm install --save fastify-helmet
```

Then, in a Fastify web application, register the `fastify-helmet` plugin and provide it a configuration object that includes any of the Helmet-supported security headers:

```js
const fastify = require("fastify")();
const helmet = require("fastify-helmet");

fastify.register(helmet, {
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
    },
  },
});

fastify.listen(3000, (err) => {
  if (err) throw err;
});
```

By registering the `fastify-helmet` plugin without any configuration, the following default security headers and their values will be set:

```json
{
  "x-dns-prefetch-control": "off",
  "x-frame-options": "SAMEORIGIN",
  "x-download-options": "noopen",
  "x-content-type-options": "nosniff",
  "x-xss-protection": "0"
}
```
