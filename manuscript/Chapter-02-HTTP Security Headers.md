# HTTP Security Headers

## HTTP Strict Transport Security

HTTP Strict Transport Security, also known as HSTS, is a protocol standard to enforce secure connections to the server via HTTP over SSL/TLS. HSTS is configured and transmitted from the server to any HTTP web client using the HTTP header _Strict-Transport-Security_ which specifies a time interval during which the browser should only communicate over an HTTP secured connection (HTTPS).

> Tip
>
> When a _Strict-Transport-Security_ header is sent over an insecure HTTP connection the web browser ignores it because the connection is insecure, to begin with.

In future requests, after the header has been set, the browser consults a _preload_ service, such as [that of Google's](https://hstspreload.org/), to determine whether the website has opted in for HSTS.

### The Risk

The risk that may arise when not communicating over a secure HTTPS connection is that a malicious user can perform a Man-In-The-Middle (MITM) attack and down-grade future requests to the webserver to use an HTTP. Once an HTTP connection is established, a malicious attacker can see and read all the data that flows through.

> Interesting fact:
> The [original HSTS draft](https://tools.ietf.org/html/rfc6797) was published in 2011 by Jeff Hodges from PayPal,
> Collin Jackson from Carnegie Mellon University, and Adam Barth from Google.

A website that uses HTTPS may still create insecure HTTP requests which end users wouldn't suspect but expose end-users to MITM attack security concerns.

In the following flow diagram, _Figure 1-1_, we can see an example scenario where the server returns an HTML file for the login page to the browser, which includes some resources that are accessible over HTTP (`http://cdn.server.com/images/submit.png`), like the submit button's image.

If an attacker can perform a Man-In-The-Middle attack and "sit on the wire" to listen and sniff any un-encrypted traffic that flows through, then they can access and read those HTTP requests which may include sensitive data. Even worse scenarios may include HTTP resources set for POST or PUT endpoints where actual data is being sent and can be sniffed.

![Figure 1-1 - Visualizing HTTPS MITM Attack](Figure1-1-VisualizingHTTPSMITMAttack.png)

### The Solution

When web servers want to protect their web clients through a secured HTTPS connection, they need to send the _Strict-Transport-Security_ header with a given value which represents the duration of time in seconds for which the web client should send future requests over a secured HTTPS connection.

e.g. to instruct the browser to upgrade all requests sent to the server to HTTPS for the next hour:

```
Strict-Transport-Security: max-age=3600
```

### Helmet Implementation

To use Helmet's HSTS library we need to download the npm package and we will also add it as a package dependency to the Node.js project we're working on:

```bash
npm install helmet --save
```

Let's set up the `hsts` middleware to indicate to a web client, such as a browser, that it should only send HTTPS requests to our server's hostname for the next month:

```js
const helmet = require("helmet");

// Set the expiration time of HTTPS requests to the
// server to 1 month, specified in milliseconds
const reqDuration = 2629746000;

app.use(
  helmet.hsts({
    maxAge: reqDuration,
  })
);
```

In the above snippet, we instruct the Express `app` to use the `hsts` middleware and respond to all requests with the `Strict-Transport-Security` header set.

Note that if for any reason the browser receives multiple HSTS header directives it will only respect and enforce the policy based on the first one sent.

It is common for web servers to have sub-domains to fetch assets from or make REST API calls to, in which case we would also like to protect them and enforce HTTPS requests. To do that, we can include the following optional parameter to the `hsts` options object:

```js
includeSubDomains: true;
```

> Tip
>
> If it is necessary to instruct the browser to disable the _Strict-Transport-Security_ a server can respond with this header's `max-age` set to `0` which will result in the browser expiring the policy immediately and enable access over an insecure HTTP connection.

### Exercise

The following exercise shows a practical example of using HTTP Strict Transport Security (HSTS),
as a browser security control to allow only HTTPS-enabled resources to be fetched from
the primary domain for a website.

#### Requirements

We will be using [Heroku](https://www.heroku.com/) as our hosting platform for an Express Node.js application, as we will need both an HTTPS-enabled hosting, as well as an HTTP hosting to switch on and off the HTTP Strict Transport Security header.

#### Source code

Obtain the source code from the official GitHub repository at: https://github.com/lirantal/nodejssecurity-headers-hsts.

Once you cloned the repository, the overall projects file structure that you should be aware of is:

1. An express server in `server.js`
2. A handlebars template in `views/home.handlebars` which serves as an example website
3. A `public` directory which has an image that we will use

#### Deployment

You'll need a Heroku account to deploy the Express web app there.

1. Sign-up for a free Heroku account, and have the `heroku` CLI installed.
2. `npm install` all the dependencies in the project.
3. Create a Heroku Node.js project
4. Login from the CLI using: `heroku login`
5. Using the `heroku` CLI create a new project, such as: `heroku git:remote -a hsts-express-example` to instantiate a git remote for the project, assuming `hsts-express-example` is the name of the heroku project name you used in step (3).
6. Deploy the app using `git push heroku master`

At this point, it should be available as both HTTPS and HTTP endpoints, such as:

- https://hsts-express-example.herokuapp.com/
- http://hsts-express-example.herokuapp.com/

#### Exercise 1

Once the Express app is deployed, try to access it: `https://hsts-express-example.herokuapp.com/`

X> ## Quiz time!
X>
X> What is special about the request to load the static unplash image (http://hsts-express-example.herokuapp.com/harley-davidson-zGzXsJUBQfs-unsplash.jpg) ?
X>a) It is originally an HTTP request
X>b) This request is upgraded to an HTTPS request
X>c) Nothing special about this HTTP request
X>
X>The correct answer is both A and B.
X>
X> Are you seeing anything going wrong with the network requests? What is not working with the favicon?
X>
X>a) The favicon is not loading because it's a bigger image than it should be.
X>b) The favicon is not loading due to the HSTS security header.
X>c) The favicon is not loading because the web server is misconfigured.
X>
X>The correct answer is B. The favicon is not loading because it is only served from an HTTP domain, but the HSTS security header is upgrading all requests to be HTTPS and so it fails to load.

Did you look at the network tab in the browser's DevTools? What did you find?

You should notice a few things happening:

- The main request to the page `https://hsts-express-example.herokuapp.com/` replies with a `Strict-Transport-Security` security header.
- The request to load the image `http://hsts-express-example.herokuapp.com/harley-davidson-zGzXsJUBQfs-unsplash.jpg` gets an internal browser redirect to its HTTPS version because the HSTS version does just that - it upgrades all requests to their HTTPS counterpart to load them securely.
- The favicon from `http://http.rip/favicon.ico` is blocked from being loaded.

X> ## Quiz time!
X>
X>
X> Update the expiration time of the HSTS setting to 0 (zero):
X>`js X> httpApp.use( X> helmet.hsts({ X> maxAge: 0, X> }) X>); X>`
X>
X>What changed?
X>
X>a) Strict-Transport-Security is set but with an expiration time of 0 which disables it.
X>b) The unsplash image is loaded from HTTP directly, without any redirect
X>c) The favicon is fetched and displayed for the website
X>
X>The correct answers are A, B, and C.

### Debugging HSTS settings in Chrome

As you experiment with the HTTP Strict Transport Security header, you may be setting it on localhost served pages, which could end up as a footgun due to HTTP-only pages being forcefully redirected to HTTPS.

#### Reviewing Chrome's HSTS settings

If such an issue occurs and you wish to review your Chrome setup of HSTS settings and manually include or exclude domains from the HSTS list, navigate to `chrome://net-internals/#hsts` in Chrome's address bar and update as necessary.

It should look like the image depicted in Figure 1-2 below:

![Figure 1-2 - Chrome's internal HSTS configuration](Figure1-2-ChromeinternalHSTSconfiguration.png)

#### Clear cache

Sometimes, no localhost entry will exist on the HSTS internal configuration for Chrome, yet a forceful redirect to HTTPS will still take place.

To ensure you clear the cache, do as follows:

1. Navigate to the localhost domain at `http://localhost`
2. Open DevTools by pressing `CTRL+SHIFT+I` or `F12`
3. Locate the address bar's reload page icon and right-click it. In the menu that opens up select `Empty Cache and Hard Reload`

Reloading the localhost website over HTTP, such as `http://localhost:3000` should now work as expected without an HTTPS redirect.

## X Frame Options

The [X-Frame-Options](http://tools.ietf.org/html/7034) HTTP header was introduced to mitigate an attack called Clickjacking. It allows an attacker to disguise page elements such as buttons, and text inputs by hiding their view behind real web pages which render on the screen using an iframe HTML element or similar objects.

> Deprecation notice
> The X-Frame-Options header was never standardized as part of an official specification but many of the popular browsers today still support it.
> Its successor is the Content-Security-Policy (CSP) header which will be covered in the next section and one should focus on implementing CSP for newly built web applications.

### The Risk

The [Clickjacking](https://owasp.org/www-community/attacks/Clickjacking) attack, also known as UI redressing, is about misleading the user to perform a seemingly naive and harmless operation while in reality, the user is clicking buttons that belong to other elements, or typing text into an input field which is under the attacker's control.

Common examples of employing a Clickjacking attack:

1. If a bank or email account website doesn't employ an `X-Frame-Options` HTTP header, then a malicious attacker can render them in an iframe, and place the attacker's input fields on the exact location of the bank or email website's input for username and password and record your credentials information.
2. A web application for video or voice chat that is insecure can be exploited by this attack to let the user mistakenly assume they are just clicking around on the screen or playing a game, while in reality, the series of clicks is actually turning on your webcam.

### The Solution

To mitigate the problem, a web server can respond to a browser's request with an `X-Frame-Options` HTTP header which is set to one of the following possible values:

1. `DENY` - Specifies that the website can not be rendered in an iframe, frame, or object HTML elements.
2. `SAMEORIGIN` - Specifies that the website can only be rendered if it is embedded on an iframe, frame, or object HTML elements from the same domain the request originated from.
3. `ALLOW-FROM <URI>` - Specifies that the website can be framed and rendered from the provided URI. It is important to note that you can't specify multiple URI values, but are limited to just one.

A few examples to show how this header is set are:

```
X-Frame-Options: ALLOW-FROM http://www.mydomain.com
```

and

```
X-Frame-Options: DENY
```

T> Beware of Proxies
T>
T> Web proxies are often used as a means of caching and they natively perform a lot of header manipulation.
T>
T> Beware of proxies that might strip off this or other security-related headers from the response.

### Helmet Implementation

With Helmet, implementing this header is as simple as requiring the `helmet` package and using Express's `app` object to instruct Express to use the `xframe` middleware provided by Helmet.

To set the `X-Frame-Options` to completely deny all forms of embedding:

```js
const helmet = require("helmet");

app.use(
  helmet.frameguard({
    action: "deny",
  })
);
```

Similarly, we can allow frames to occur only from the same origin by providing the following options object:

```js
{
  action: "sameorigin";
}
```

Or to allow frames to occur from a specified host:

```js
{
  action: 'allow-from',
  domain: 'https://mydomain.com'
}
```

### Exercise

The following exercise shows a practical clickjacking attack by a malicious party. In this example, the attacker deploys a website they control with a hidden iframe. The website is rendered in an iframe is an innocent, third-party website. The attacker's aim is to hijack any clicks made by unsuspecting users on that website.

#### Requirements

Node.js and npm are expected to be available in your development environment as we will run this exercise locally.

Note: In this exercise, there's no strict need for serving the web pages content over HTTPS.

#### Source code

Obtain the source code from two official GitHub repositories:

- https://github.com/lirantal/nodejssecurity-headers-xframe-malicious - which serves the contents of a malicious website that embeds a remote iframe in an attempt to trick the user to click on
- https://github.com/lirantal/nodejssecurity-headers-xframe-innocent - which serves the contents of an innocent website. In our example, this serves as a Twitter profile card.

Once you cloned both repositories locally we are ready to run both servers.

#### Deployment

To run this exercise we will begin by installing all the dependencies for each npm project and then run the Express servers:

In each directory where the projects are cloned:

1. `npm install` all the dependencies
2. Run `npm start` in two terminal windows so we can have the Express servers run in parallel

The servers will require that you have ports 3000 and 3001 available to bind to by default. Otherwise, you may provide a `PORT` environment variable to each web server project to configure a different local port.

#### Exercise 1

Load up the malicious website by navigating to `http://localhost:3000`.

You will be presented with a website asking you to sign-up for a React developer newsletter. Would you?

![Exercise1](Figure1-3Excercise1.png)

D> ## Quiz time!
D>
D>
D> The invite to join this React newsletter is quite tempting. Did you click it? What happened?

It looks like clicking the `Sign up!` button on this website doesn't do what you hoped it would.

![Exercise1 - Interacting with a malicious website](Figure1-4Exercise1.png)

#### Exercise 2

You're not sure what was going on, right?
What if you could see the actual element you clicked on?

Add `reveal=1` query parameter to the website, such as: `http://localhost:3000?reveal=1` and the iframe that has been loaded in the background will be revealed in 50% opacity so you can see how it renders on the screen and perfectly aligns with the "Sign up!" button.

![Exercise2 - Interacting with a malicious website](Figure1-5Exercise2.png)

#### Exercise 3

Open `views/home.handlebars` and update the iframe URL to some other websites, maybe Twitter itself but for real this time?

D> ## Quiz time!
D>
D> What happened when you changed the iframe URL to something else?
D> Can you try and find a website that should be secure but allows rendering in an iframe?

#### Exercise 4

Ok, let's fix things.

Open `./malicious-website/views/home.handlebars` and update the iframe URL to `http://localhost:3001/html/twitter.html` and make sure the other innocent web server, which we cloned earlier, is running.

Browse to the malicious website again. Did anything change?

For the innocent website, let's make sure that we update it to disallow rendering itself as an iframe with the help of Helmet's `frameguard` middleware:

```js
const helmet = require("helmet");
app.use(helmet.frameguard({ action: "deny" }));
```

Refresh the malicious website and note the changes.

## Content Security Policy

As reviewed before with the X-Frame-Options header, there are many attacks related to content injection in the user’s browser. These include Clickjacking attacks or other forms of attacks such as Cross-Site-Scripting (XSS).

I> What is an XSS?
I>
I> A Cross-site scripting, or XSS for short, is a type of security attack in which a user can inject JavaScript, or other types of scripts (for example injecting CSS, or HTML) to trigger the execution of them by the context interpreter, such as the browser.

Another improvement to the previous set of headers we reviewed so far is a header that can tell the browser which content to trust. This allows the browser to prevent attempts of content injection that are not trusted in the policy defined by the application owner.

With a [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/Security/CSP/Introducing_Content_Security_Policy) (CSP) it is possible to prevent a wide range of attacks, including Cross-site scripting and other content injections. The implementation of a CSP renders the use of the X-Frame-Options header obsolete.

### The Risk

Using a Content Security Policy header will prevent and mitigate XSS and other injection attacks. Examples of some of the issues it can prevent by setting a CSP policy:

- Inline JavaScript code specified with `<script>` tags, and any DOM events which trigger JavaScript execution such as `onClick()` etc.
- Inline CSS code specified via a `<style>` tag or attribute elements.

### The Solution

With CSP allowlists, we can allow many configurations for trusted content, and as such the initial setup can grow to a set of complex directives.

Let's review one directive called _connect-src_. It is used to control which remote servers the browser is allowed to connect to via XMLHttpRequest (XHR), or `<a>` elements. Other script interfaces that are covered by this directive are: `Fetch`, `WebSocket`, `EventSource`, and `Navigator.sendBeacon()`.

Acceptable values that we can set for this directive:

- _'none'_ - not allowing remote calls such as XHR at all.
- _'self'_ - only allow remote calls to our domain (an exact domain/hostname - sub-domains aren't allowed).

An example for such content security policy being set is the following directive which allows the browser to make XHR requests to the website's own domain and Google's API domain:

```
Content-Security-Policy: connect-src 'self' https://apis.google.com;
```

Another directive to control the allowlist for JavaScript sources is called _script-src_.
This directive helps mitigate Cross-Site-Scripting (XSS) attacks by informing the browser which sources of content to trust when evaluating and executing JavaScript source code.

_script-src_ supports the _'none'_ and _'self'_ keywords as values and includes the following options:

- _'unsafe-inline'_ - allow any inline JavaScript source code such as `<script>`, and DOM events triggering like `onClick()` or `javascript:` URIs. It also affects CSS for inline tags.
- _'unsafe-eval'_ - allow execution of code using `eval()`.

For example, a policy for allowing JavaScript to be executed only from our own domain and from Google's, and allows inline JavaScript code as well:

```
Content-Security-Policy: script-src 'self' https://apis.google.com 'unsafe-inline'
```

Note, the `'unsafe-inline'` directive refers to a website's own JavaScript sources.

A full list of supported directives can be found on the [CSP policy directives page on MDN](https://developer.mozilla.org/en-US/docs/Web/Security/CSP/CSP_policy_directives) but let's cover some other common options and their values.

- _default-src_ - where a directive doesn't have a value, it defaults to an open, non-restricting configuration. It is safer to set a default for all of the un-configured options and this is the purpose of the _default-src_ directive.
- _script-src_ - a directive to set which script sources we allow to load or execute JavaScript from. If it's set to a value of _'self'_ then it will only allow sources from our own domain. Also, it will not allow inline JavaScript tags, such as `<script>`. To enable those, add `'unsafe-inline'` too.

> On implementing CSP
> It should also be noted that the CSP configuration needs to meet the implementation of your web application architecture. If you
> deny inline `<script>` blocks then your R&D team should be aware and well prepared for this as otherwise, this will be breaking features and functionality across code that depends on inline JavaScript code blocks.

### Helmet Implementation

Using Helmet we can configure a secure policy for trusted content.
Due to the potential for a complex configuration, we will review several different policies in smaller blocks of code to easily explain what is happening when we implement CSP.

The following Node.js code will add Helmet's CSP middleware on each request so that the server responds with a CSP header and a simple security policy.

We define an allowlist in which JavaScript code and CSS resources are only allowed to load from the current origin, which is the exact hostname or domain (no sub-domains will be allowed):

```js
const helmet = require("helmet");

app.use(
  helmet.contentSecurityPolicy({
    directives: {
      scriptSrc: ["'self'"],
      styleSrc: ["'self'"],
    },
  })
);
```

It is important to remember that if no default policy is specified then all other types of content policies are open and allowed, and also some content policies simply don't have a default and must be specified to be overridden.

Let's construct the following content policy for our web application:

- By default, allow resources to load only from our own domain origin, or from our Amazon CDN. The `defaultSrc` refers to all script types sources, such as CSS, iframes, fonts, etc.
- JavaScript sources are restricted to our own domain and Google's hosted libraries domain so we can load AngularJS from Google.
- Because our web application doesn't need any kind of iframes embedding we will disable such objects (refers to `objectSrc` and `childSrc`).
- Forms should only be submitted to our own domain origin.

```js
var helmet = require("helmet");

app.use(
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'", "https://cdn.amazon.com"],
      scriptSrc: ["'self'", "https://ajax.googleapis.com"],
      childSrc: ["'none'"],
      objectSrc: ["'none'"],
      formAction: ["'none'"],
    },
  })
);
```

### Gradual CSP Implementation

Your Content Security Policy will grow and change as your web application grows too. With the many varied directives, it could be challenging to introduce a policy all at once so instead of touch-and-go enforcement, strive for an incremental approach.

The CSP header has a built-in directive that helps in understanding how your web application makes use of the content policy. This directive is used to track and report any actions performed by the browser that violate the content security policy.

It's simple to add to any running web application:

```
Content-Security-Policy: default-src 'self'; report-uri https://mydomain.com/report
```

> Note that the semicolon is added to end the content security policy directives, and begin a new `report-uri` directive.

Once added, the browser will send a POST request to the URI provided with a JSON format in the body for anything that violates the content security policy of only serving content from our own origin.

With Helmet's `csp` middleware this is easily configured:

```js
const helmet = require("helmet");

app.use(
  helmet.csp({
    directives: {
      defaultSrc: ["self"],
      reportUri: "https://mydomain.com/report",
    },
  })
);
```

Another useful configuration for Helmet when we are still evaluating a Content Security Policy is to instruct the browser to only report on content policy violations and not block them:

```js
const helmet = require("helmet");

app.use(
  helmet.csp({
    directives: {
      defaultSrc: ["self"],
      reportUri: "https://mydomain.com/report",
    },
    reportOnly: true,
  })
);
```

## Referer and Referrer Policy

When users browse through web pages, the browser may set a request header called `Referer` in certain conditions. This `Referer` header is often used by backend servers to track users' behavior for analytics and other means.

How does `Referer` look like in an HTTP request?

If we were to search for `wikipedia` on Google, and click on the Wikipedia search result on the page, we could then see the `Referer` header set as such:

![Figure 1-6: DevTools showing the Referer header set from a web page](Figure1-6Referer1.png)

What if a web page had stored sensitive information in a URL such as an account ID as part of the URL, or other sorts of sensitive information about the system? If a link on that page is then visited, and the browser sets the `Referer` header as it would normally, that could lead to sensitive information leakage.

This is where the `Referrer Policy` header comes in. This header, when set by a web server, instructs the browser whether to populate the `Referer` header when navigating out of that web page and into a new one.

T> An insecure way of using a Referer header?
T> Because the `Referer` header is set on the client-side, and may be abused, it shouldn't be trusted as a source of truth and its integrity should be considered minimal.
T> This is why browsers will remove the `Referer` header when browsing from an HTTPS website to an HTTP website.
T> Reading reference on this topic would be [Referer Spoofing](https://en.wikipedia.org/wiki/Referer_spoofing) on Wikipedia.

### Referrer Policies

The `Referrer Policy` header can set one of the following policies that instruct the browser's behavior when navigating off the page:

- no-referrer
- no-referrer-when-downgrade
- origin
- origin-when-cross-origin
- same-origin
- strict-origin
- strict-origin-when-cross-origin
- unsafe-url

Let's review each of these values.

#### no-referrer

Instruct the browser to never set a `Referer` header at all, for any links related to requests from this web page.

#### no-referrer-when-downgrade

If there's a security downgrade in the form of making requests from an HTTPS website to an HTTP, then the browser doesn't set the `Referer` header.

We mentioned before in the above hint that this is indeed the default behavior that browsers follow if a referrer policy is unset, or invalid.

#### origin

Instead of sending the full URL - the origin, path, and query parameters of the current page being navigated from, the browser will only send the origin, such as `https://www.google.com` and nothing beyond that in the URL.

#### origin-when-cross-origin

As the name implies, only the origin is sent to any requests the browser makes to navigate off the page, when those addresses match a cross-origin. Otherwise, when requests are made to URLs of the same origin (as complies with the same-origin policy), the default behavior of setting the `Referer` header to the current URL is followed.

#### same-origin

The current URL is set for the `Referer` header to any requests that are considered same-origin, otherwise, it isn't set at all.

#### strict-origin

As we've seen in other policies now - if there's a security downgrade in the form of making requests from an HTTPS website to an HTTP, then the browser doesn't set the `Referer` header at all. When requests are kept in the same origin, only the origin is set as the `Referer` value.

#### strict-origin-when-cross-origin

This policy setting is a bit more nuanced:

- If there's a security downgrade in the form of making requests from an HTTPS website to an HTTP, then the browser doesn't set the `Referer` header at all
- If the request is made to an HTTPS cross-origin address, then only the origin is set for the `Referer` header.
- If the request is made to the same origin, then the full URL is set.

#### unsafe-url

This is the least secure option, which always sets a value for the `Referer` header and could lead to a sensitive information leak.

T> Can you specify a Referrer Policy in HTML?
T> Yes you can!

Simlar to other security headers, such as the Content Security Policy, you can define the browser's behavior with regards to the `Referer` header by using HTML meta tags on the page.

For example:

```html
<meta http-equiv="Referrer-Policy" content="strict-origin-when-cross-origin" />
```

If you are using a Content Security Policy to set trusted content policies for the browser, and have used the `referrer` directive, then this is now deprecated and has been superseded by the `Referrer Policy` header as a dedicated means of conveying the same information.

### Helmet Implementation

Using Helmet, we can configure the desired referer policy:

```js
const helmet = require("helmet");

app.use(
  helmet.referrerPolicy({
    policy: "no-referrer",
  })
);
```

## X XSS Protection

The HTTP header _X-XSS-Protection_ is used by IE8 and IE9 and allows toggling on or off the Cross-Site-Scripting (XSS) filter capability that is built into the browser.

Turning XSS filtering on for any IE8 and IE9 browsers rendering your web application requires the following HTTP header to be sent:

```
X-XSS-Protection: 1; mode=block
```

With Helmet, this protection can be turned on using the following snippet:

```js
const helmet = require("helmet");

app.use(helmet.xssFilter());
```

## X Content Type Options

When browsers fetch remote sources of content, such as JavaScript or images, they are instructed using the `Content-Type` header on the type of content.

For example, when a PDF content type is fetched by the browser, the server hints the browser about it by setting the following header: `Content-Type: application/pdf`.

These content types are standardized by the IANA organization as MIME types, and a [full list of common MIME types can be seen here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Common_types).

### Risk

What happens when the browser is instructed an incorrect MIME type for a content, or not at all entirely? In such a case, the browser will attempt to guess the content type by reading and interpreting the content data. This action is referred to as _MIME Sniffing_.

> More information on MIME Sniffing can be found in the official [MIME Sniffing standard](https://mimesniff.spec.whatwg.org/).

The purpose of this header is to instruct the browser to avoid guessing the web server's content type which may lead to an incorrect render than that which the webserver intended.

The _X-Content-Type-Options_ HTTP header is used by IE, Chrome, and Opera and is used to mitigate a MIME-based attack.

An example of setting this header:

```
X-Content-Type-Options: nosniff
```

Helmet's implementation:

```js
const helmet = require("helmet");

app.use(helmet.noSniff());
```
