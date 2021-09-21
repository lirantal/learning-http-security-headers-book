# What's next?

## Establish a CSP and Security Headers standard

Adopt new browser and HTTP security standards and set a plan to migrate from old HTTP headers.

### X-Frame-Options

We previously reviewed the benefits of using the `X-Frame-Options` as a response HTTP header in helping address click-jacking security vulnerabilities in web applications. That said, practices evolve and browsers rapidly adopt new standards and mechanisms. For example, the `ALLOW-FROM` value for the X Frame Options header has been deprecated and is discouraged from being used because modern browser versions don't support it anymore.

As a migration path, the Content Security Policy standards create a way to adapt to such new standards. One of which is, CSP's `frame-ancestors` directive. For example, setting its value to `'none'` should be compatible with `X-Frame-Options` setting of `DENY` value. A more complete example of the Content Security Policy in action for click-jacking security controls is:

```
Content-Security-Policy: frame-ancestors 'none';
```

The above CSP will disallow any URLs of embeddable content in `iframe`, `object`, and other HTML elements which are part of the `frame-ancestors` policy.

Do note however, that older browsers may not respect Content-Security-Policy and its directives and as such, you may actually cause a degraded security status. To avoid such a problem, consult your supported browser matrix requirements, employ both old and new headers to ensure all bases are covered, until possible to deprecate older security controls that are no longer valid.

### X-XSS-Protection

Similar to the case with the `X-Frame-Options` HTTP header, the `X-XSS-Protection` header is considered deprecated completely and should mandate that you establish and roll out a Content Security Policy HTTP header instead.

It's still useful to keep as a header if you are targeting older browsers, but otherwise, note that Chrome and Edge removed their XSS auditor, and Firefox isn't planning on implementing support for X-XSS-Protection.

Following is the [browser compatibility matrix for the header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection) as listed on MDN:

![Figure 3-1: Partial screenshot of the MDN X-XSS-Protection header page (https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection)](Figure3-1Next1.png)

### Summary

Security controls will regularly need fine-tuning and keeping up to date as newer and modern technologies evolve around them. This isn't to say that the two security headers we studied and reviewed here are useless. On the contrary. We need to understand in which situations they are still relevant and establish a migration path to newer standards that will eventually replace them.

## Monitor your web application

We learned how to increase security using web controls such as HTTP headers, but to further ensure that they are kept in check we need to monitor them.

We learned how to increase security using web controls such as HTTP headers, but how do we ensure that we're always up to date with security controls? How do we ensure there isn't a regression next week where headers are removed from an HTTP response? The problem can get a lot harder even if you're working in a rich microservices environment, and needing to account for more than a few services.

### Monitoring and Shifting-left

Shift-Left is a concept in which we refer to moving activities to be as early as possible in the software development lifecycle.

We would ideally want to shift left as much of the monitoring activities as possible, so we can ensure that problems are detected earlier in the process rather than after the fact.

We reviewed some tools in which we can easily create a CI integration during the build process. For example, we can leverage a full WebPageTest integration for both its performance and security insights by triggering an API call upon a successful website deployment to run an end-to-end build.

Furthermore, we can use command-line tools such as Check My Headers and others to validate that server response are indeed conforming to a policy. This helps us shift left in application security testing and find issues earlier in the software development lifecycle.

## Other browser security headers and controls

The web is an evolving standard and as such, new security controls would be introduced. We should keep an eye on them!
Embrace and prepare for privacy, feature controls, and future headers such as `Referrer-Policy`, `Feature-Policy`, `Origin-Policy`, `Integrity`, `Accept-CH`, and `Clear-Site-Data`.

As the web evolves, it creates new standards for us to adopt. This also applies to new HTTP headers and we will quickly review a bunch of them here as your future steps in establishing a wider range of headers.

## Referrer-Policy

Embrace and prepare for privacy-related policies using `Referrer-Policy`, which instructs the browser when and how much information to provide when setting a `Referer` header as users navigate from an existing web page.

Some example values for Referrer Policy are:

```
Referrer-Policy: no-referrer
Referrer-Policy: origin-when-cross-origin
Referrer-Policy: same-origin
```

The default value set by the browser is `no-referrer-when-downgrade`, however, a more recommended setting will be one of the strict-origin options, such as `strict-origin-when-cross-origin`. That setting ensures that complete referrer information is sent when requests are kept to the same origin and so are bound to the same web application context. Then only sending the origin (not the full path) to any requests that are kept within the same secure HTTPS level, and nothing otherwise.

The browser support matrix as to the date of writing this is as follows:

![Figure 3-2: Screenshot of the browser version support for the Feature-Policy HTTP header as provided in the MDN website for it: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Feature-Policy](Figure3-2Next2.png)

Helmet doesn't yet support the `Feature-Policy` header. However, you may define it in a Node.js web application using one of the following ways:

1. Set that HTTP header as part of the web application framework. The following is an example of using Express to set it for all responses:

```js
app.use(function (req, res, next) {
  res.setHeader("Feature-Policy", "geolocation 'none'");
  return next();
});
```

2. Use the independent [feature-policy](https://snyk.io/advisor/npm-package/feature-policy) module on npm.

### Summary

As the web evolves, security controls are evolved along with it. For example, a security header that increases user privacy is [Clear-Site-Data](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Clear-Site-Data) which aims at minimizing the scope of data at rest for a website.

Gradually implement more HTTP security headers to increase the controls you have for your web application, and create mitigation points for vulnerabilities.

## Educational resources

Where-as this learning experience isn't geared at being a comprehensive list of all available security headers, you should refer to other web resources such as [Mozilla's Developer Network](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers) and the [W3C specs](https://www.w3.org/standards/) to keep up to date.

In particular, I want to recommend the following topics to add a lot of relevant context and gaining an edge in understanding web security:

- Cross-Origin topics, and particularly [Cross-Origin-Resource-Sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).
- [Sub-resource Integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity) policies.
- [Cross-site Request Forgery](https://infosec.mozilla.org/guidelines/web_security#csrf-prevention) and related forms of tokenization.
- Understanding how [Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies) work and spec updates such as `SameSite` attribute.
