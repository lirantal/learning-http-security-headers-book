# Testing for Security Headers

## The State of HTTP Security

What is the state of HTTP security today for the web? Are most people enabling HTTPS? Luckily, there's an open project that tracks this, and more, in order to gain and share these insights.

The web, primarily runs on HTTP, but to ensure the security, integrity, and privacy of end-to-end connections, clients communicate over a secure HTTP known as HTTPS.

The importance of a secure communications channel shouldn't be undervalued. Instead, it should be a standard for any size of web applications, whether static or dynamic and indeed HTTPS is more prevalent than ever.

An important push for HTTPS has been made by browsers themselves, such as Chrome's continuous attempts to discourage the use of HTTP by portraying any such websites as potentially dangerous.

A prime example of that is Chrome's recent [hardened policy about mixed content](https://security.googleblog.com/2019/10/no-more-mixed-messages-about-https_3.html) which actively blocks HTTP requests, and it follows prior actions taken to increase the importance of security aspects of the web, such as:

- Clearer indications of a website's security based on [green lock icon in the address bar](https://blog.mozilla.org/security/2017/01/20/communicating-the-dangers-of-non-secure-http/)
- A dedicated [Security panel](https://developers.google.com/web/updates/2015/12/security-panel) on Chrome's DevTools

![Figure 2-1: Chrome DevTools new Security panel](Figure2-1StateOfHTTP1.png)

### The HTTP Archive

The [HTTP Archive](https://httparchive.org/) is an important initiative by web activists that is tracking various aspects and traits of how the web evolves over time. The projects in the HTTP archive are [open source](https://github.com/HTTPArchive/httparchive.org) and managed by a community of developers.

Some of the well known reports that have been made public and online from the HTTP Archive are:

- [State of the Web](https://httparchive.org/reports/state-of-the-web) - tracks the adoption of web technologies and growing web standards across websites. It reports on data points such as `Total Requests`, `Pages with Vulnerable JavaScript libraries`, and the prevalence of `HTTP/2 Requests` in websites, in the aim of identifying trends.
- [State of JavaScript](https://httparchive.org/reports/state-of-javascript) - tracks the overall impact of JavaScript in a website, with identifying data points such as the size of JavaScript libraries in a website, the amount of JavaScript requests and the boot-up time which indicates the amount of CPU time each script consumes on a webpage.
- [Accessibility Report](https://httparchive.org/reports/accessibility) - tracks an overall accessibility score, as noted by Chrome's Lighthouse tool, and other accessibility traits and standards such as the use of `Image Alt` attributes.

I> Chrome Lighthouse
I>
I> Lighthouse is an open-source browser automation tool which helps in auditing a web page for performance, security, accessibility and other metrics. It also provides an overall score and recommendations for improving a web page.

The [data for all HTTP Archive reports](https://httparchive.org/faq#how-do-i-use-bigquery-to-write-custom-queries-over-the-data)) is made available via Google’s BigQuery for anyone to examine. It is compiled by analyzing Alexa’s top 1 million websites, in bi-weekly scans, using the open source project and the online web performance tool [WebPageTest](https://webpagetest.org).

#### HTTPS Requests

Using the HTTP Archive as a tool, we can see the growth in trend of _secure by default_ with regards to HTTPS adoption by websites.

I> Secure by default
I>
I> A secure by default approach refers to using or initializing a component with safe and secure default values, unless explicitly stated otherwise.

The earliest data point is January 2016, which states a 24% of desktop websites using HTTPS, and whooping 87.7% by August 2020 across the same category.

![Figure 2-3: HTTP Archive's State of the Web - HTTPS Requests](Figure2-1StateOfHTTP2.png)

#### Secure Hosting

With the growth of HTTPS, static website hosting platforms have adjusted and adopted similar standards, and help push towards a more secure web.

All of the following platforms for deploying and hosting your websites will serve your content over HTTPS:

- Vercel
- Netlify
- Google's Firebase
- Heroku

This helps strengthen the ubiquity of HTTPS and its accessibility for small and large websites alike.

[Let's Encrypt](https://letsencrypt.org) had certainly contributed a lot to a secure web by making certificates affordable (completely free).

## WebPageTest

[WebPageTest](https://webpagetest.org) is one of the most popular tools around the Web Performance community to provide page speed insights, bottlenecks breakdown reports, and further information when measuring a website's performance.

It is an [open source project](https://github.com/WPO-Foundation/webpagetest) that is maintained by long-time Google's software engineer [Patrick Meenan](https://github.com/pmeenan). Many leverage the project to run their performance tests in a hosted environment, where they can provide their internal resources to run end-to-end or periodical smoke test scans, to keep an eye on the quality of their web assets.

I> Smoke test
I>
I> Smoke testing is a pattern of running a small sub-set of tests to ensure a minimal yet vital and critical flow or business capability.

A relatively recent addition that was introduced to WebPageTest (May 2020) now provides users with security insights as to the status of HTTP security headers and detection of vulnerable JavaScript libraries that are rendered in scanned web pages.

### Running a scan

Head over to https://webpagetest.org and enter the URL for a web page of your preference. For our demo purposes, we'll use the Fox News website `https://www.foxnews.com/` as a website to scan and see what security information can we find, to further improve the website's security posture.

You may choose to configure other settings for performance, such as tweaking the location origin for running the test, specifying a browser type or maybe even a mobile device, and many other fine tunings.

However, we won't be needing any of the special configurations to get a security score so go ahead and hit the **START TEST** button on the right once you've entered a URL:

![Figure 2-2: Scanning a website with WebPageTest](Figure2-2WebPageTest1.png)

### Test results

Testing will take a few seconds, perhaps even up to a couple of minutes as all the test requests are queued up in the publicly available nodes for WebPageTest.

Once testing is complete, WebPageTest presents the main test results which include:

- Top-page scores
- Test results summary for browser performance metrics such as First Byte, First Contentful Paint, Total Blocking Time, Document Complete, and Fully Loaded. These performance metrics are, however, out of the scope for us.
- Waterfall for all the requests. Should be familiar and similar to the browser's DevTools.

Here is how a test result looks like for `https://www.foxnews.com`:

![Figure 2-2: WebPageTest test result showing top level scores](Figure2-2WebPageTest2.png)

Click on the `E` score rectangle to find out more.

This takes us to the Snyk website scanner results for JavaScript vulnerabilities and security headers from the WebPageTest scan.

We can clearly see the same score of `E` on the Snyk results, but here we also get a split-screen view of the scan. It is comprised of JavaScript libraries with vulnerabilities that were detected, as well as missing HTTP security headers in the web page's response.

![Figure 2-2: WebPageTest test shows security results for Fox News website](Figure2-2WebPageTest3.png)

#### JavaScript libraries with vulnerabilities

Let's take a closer look at each of these security insights that we received.

We can spot both [lodash](https://snyk.io/vuln/npm:lodash) and [jquery](https://snyk.io/vuln/npm:jquery) with 5 vulnerability reports among them.

Maybe that web page is not exploitable through Prototype Pollution vulnerabilities and Cross-site Scripting (XSS), or maybe it is. Why take the chance?

Remediate the security vulnerability and the low score by upgrading to the latest version of these libraries which includes a fix for the security vulnerability.

#### Security headers

To the right of the test results, we see the score status related to the HTTP security headers detected as part of the HTTP response of the web page.

WebPageTest was able to successfully detect the good practice of responding with the HTTP Strict Transport Security header. That's a great start. However, it looks like there are a bunch of other HTTP headers that we've learned about before which are missing. Some of these are showing up at the top of the list such as `X-Content-Type-Options`, and `X-Frame-Options`.

### Exercise

Now it's time to test yourself and see what scores you get on your company website, your personal blog, or your favorite website.

Scan a website and get a security score.
Do you know how to fix it?

You'll find more information about this topic in the following blog article about [website security score](https://snyk.io/blog/website-security-score-explained). Dig through and make sure you know have the skills to assess security headers for a website, and the notion of vulnerable JavaScript libraries.

### Summary

WebPageTest is an online web tool that is well known for performance testing. Small unknown fact is that relatively recently (May 2020) it received an update to also report on security status of websites. It is not to be considered as a security penetration testing tool, but rather revealing the status of HTTP security headers employed by a website and detecting vulnerable JavaScript libraries.

## Chrome's Lighthouse

Using Lighthouse we can learn how to improve our website's metrics based on insights and recommendations provided after a scan is performed.

It's right there in your Chrome's DevTools console, and chances are that you aren't utilizing it highly enough to get everything you can out of it, including the security aspects.

### Getting started

Continuing with our previous example, let’s browse over to Fox News’ website.

Once the website has loaded, launch DevTools using the `F12` keyboard key or `CMD-OPTION-I` if you're on a Mac. The location of the DevTools console might appear elsewhere in your setup, or as its own window.

Once opened, click on the `Lighthouse` tab and you should see the available categories to include in our tests.

![Figure 2-3: Chrome Lighthouse in action](Figure2-3Lighthouse1.png)

While it may not seem very obvious to begin with, but the security part is included as part of the `Best practices` category. This will reveal any vulnerable JavaScript libraries and their versions that are loaded on the current web page.

Click the `Generate report` and let it run and collect the data about the page. Finally, we'll be presented with top scores for each category.

![Figure 2-3: Chrome Lighthouse scan results](Figure2-3Lighthouse2.png)

Very clearly, the website is doing poorly on performance, but notice the best practices score isn't really high either. Let's take a further look down and check what is going on there.

We can click on the `Best Practices` score or scroll down on our own and see the results, which open with the `Trust and Safety` for the website headline:

- Links to cross-origin destinations were detected on the website, which is considered to be unsafe.

- This website includes front-end JavaScript libraries with known security vulnerabilities. 8 of them in total, and as we expand the view we can see the libraries, their versions, and the vulnerability count for each.

![Figure 2-3: Chrome Lighthouse security scan results](Figure2-3Lighthouse3.png)

### Summary

A repeated theme here from both WebPageTest in the previous chapter, as well as covering Lighthouse here as a security testing tool, is JavaScript libraries with known vulnerabilities.

Lighthouse is Google's open source browser automation and testing tool to provide you with insights about web best practices for performance, accessibility, security, and more. It's right there in your Chrome's DevTools console, and chances are that you aren't utilizing it highly enough to get everything you can out of it, including the security aspects.

We primarily focused on the importance of HTTP security headers, but other web security concerns and best practices shouldn't be overlooked.

## Check My Headers command line application

In this lesson, we'll learn how to use a command-line application to retrieve HTTP headers set for a web page.

[check-my-headers](https://github.com/UlisesGascon/check-my-headers) is a fast and simple command-line application in Node.js to ping web servers and inspect the HTTP security headers to provide a status log.

Unlike online and built-in tools, `check-my-headers` would need to be installed in your development environment, or perhaps better yet, in your continuous integration pipeline.

I> Continuous Integration
I>
I> A Continuous Integration pipeline harness automation to verify a software is building successfully, as well as functioning per an expected threshold and set of tests.

It is open source, and based on Node.js, and so if you have a JavaScript tooling environment setup then it can be easily installed.

In a modern Node.js environment we can make use of the `npx` tool to execute a one-off executable npm package.

To start a scan, we can run the following command:

```
npx check-my-headers https://example.com
```

This will yield a result as the following screenshot proposed by Ulises Gascon, the author of this tool:

![Figure 2-4: Test results of check-my-headers Node.js CLI](Figure2-4CheckMyHeaders1.png)

`check-my-headers` can also be used programmatically. As it is an npm package, it can be used as a library, in the following way:

```js
const checkMyHeaders = require("check-my-headers");

checkMyHeaders("http://example.com").then(({ messages, headers, status }) => {
  console.log(`Status code: ${status}`);
  console.log(`Messages:`);
  console.log(messages);
  console.log("Current headers:");
  console.log(headers);
});
```

The above will test the web page `http://example.com` for HTTP headers and return a promise, upon which it prints the result data of the scan to the console.

## Summary

We looked at several tools to help us find security issues in web applications:

- WebPageTest - An online web performance and security scanning tool for websites.
- Lighthouse - Browser-based web assessment tool for performance, accessibility, security, and more.
- Check My Headers CLI app - a handy command-line Node.js application to test a website's headers.

### Test yourself

Let's see how well you know the tools we reviewed.

X>
X>
X>
X>

X> ## Quiz time!
X>
X> ### WebPageTest
X> WebPageTest helps with:
X>
X>a) Testing for performance issues in websites
X>b) Testing for security issues in websites
X>c) Testing for performance and security issues in websites and giving me insights into how to fix them
X>
X>The correct answer is C.

X> ### Lighthouse
X> Lighthouse is available via Chrome DevTools and helps with:
X>
X>a) Finding performance issues
X>b) Finding security issues
X>c) Finding SEO and Web Accessibility issues
X>d) Finding issues with Progressive Web Apps
X>
X>The correct answers are A,B,C, and D.

X> ### Keeping up with security
X> What are some ways you can make sure you have no regressions in your security headers setup?
X>
X>a) Run tools like `check-my-headers` in my Continuous Integration systems to fail the build if a regression happens
X>b) In an End-to-End Continuous Integration setup I can use the WebPageTest API to schedule tests of my website and ensure the security score is the same, or better
X>c) Run a security penetration test after the web application is published
X>d) This is a manual and rigorous process that takes time, very expensive and is hard to keep up repeating effectively.
X>
X>The correct answers are A and B.

What's next?

If you'd like to keep security in check, you'd want to automate it to keep up with the scale of development. All of the above tools have APIs or integration points that you can connect to continuous integration systems.
