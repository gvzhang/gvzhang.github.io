---
title: 如何通过HTTP头加固WEB应用
categories:
 - TCP/HTTP
tags:
 - HttpHeaders
---

####敏感信息禁止缓存
- **Cache-Control**
This response header, introduced in HTTP 1.1, may contain one or more directives, each carrying a specific caching semantic, and instructing HTTP clients and proxies on how to treat the response being annotated by the header. My recommendation is to format the header as follows: cache-control: no-cache, no-store, must-revalidate. These three directives pretty much instruct clients and intermediary proxies not to use a previously cached response, not to store the response, and that even if the response is somehow cached, the cache must be revalidated on the origin server.
- **Pragma: no-cache**
For backwards-compatibility with HTTP 1.0, you will want to include this header as well. Some HTTP clients, especially intermediary proxies, still might not fully support HTTP 1.1 and so will not correctly handle the Cache-Control header mentioned above. Use Pragma: no-cache to ensure that these older clients do not cache your response.
- **Expires: -1**
This header specifies a timestamp after which the response is considered stale. By specifying -1, instead of an actual future time, you ensure that clients immediately treat this response as stale and avoid caching.

```Node.js

function requestHandler(req, res) {
	res.setHeader('Cache-Control','no-cache,no-store,max-age=0,must-revalidate');
	res.setHeader('Pragma','no-cache');
	res.setHeader('Expires','-1');
}

```
####强制使用HTTPS
- **max-age=<number of seconds>**
This instructs the browser to cache this header, for this domain, for the specified number of seconds. This can ensure tightened security for a long duration!
- **includeSubDomains**
This instructs the browser to apply HSTS for all subdomains of the current domain. This can be useful to cover all current and future subdomains you may have.
- **preload**
This is a powerful directive that forces browsers to always load your web app securely, even on the first hit, before the response is even received! This works by hardcoding a list of HSTS preload-enabled domains into the browser’s code. To enable the preloading feature, you need to register your domain with HSTS Preload List Submission, a website maintained by Google’s Chrome team. Once registered, the domain will be prebuilt into supporting browsers to always enforce HSTS. The preload directive within the HTTP response header is used to confirm registration, indicating that the web app and domain owner are indeed interested in being on the preload list.
```Node.js

function requestHandler(req, res) {
	res.setHeader('Strict-Transport-Security','max-age=31536000; includeSubDomains; preload');
}

```
####XSS过滤
To help protect users against reflective XSS attacks, some browsers have implemented protection mechanisms. These mechanisms try to identify these attacks by looking for matching code patterns in the HTTP request and response. 

Luckily, there is a way for a web app to override this configuration and ensure that the XSS filter is turned on for the web app being loaded by the browser. This is done via the **X-XSS-Protection** header. This header, supported by Internet Explorer (from version 8), Edge, Chrome and Safari, instructs the browser to turn on or off the browser’s built-in protection mechanism and to override the browser’s local configuration.

**X-XSS-Protection** directives include these:
- 1 or 0
This enables or disables the filter.
- mode=block
This instructs the browser to prevent the entire page from rendering when an XSS attack is detected.

```Node.js

function requestHandler(req, res) {
	res.setHeader('X-XSS-Protection','1;mode=block');
}

```

####控制IFRAME
Clickjacking is an attack that tricks the user into clicking something different than what they think they’re clicking. 
```HTML
<html>
  <body>
    <button class='some-class'>Win a Prize!</button>
    <iframe class='some-class' style='opacity: 0;’ src='http://buy.com?buy=toaster'></iframe>
  </body>
</html>
```
An effective way to block this attack is by restricting your web app from being framed. **X-Frame-Options**, specified in RFC 7034, is designed to do exactly that! This header instructs the browser to apply limitations on whether your web app can be embedded within another web page, thus blocking a malicious web page from tricking users into invoking various transactions on your web app. You can either block framing completely using the DENY directive, whitelist specific domains using the ALLOW-FROM directive, or whitelist only the web app’s origin using the SAMEORIGIN directive.
```Node.js

function requestHandler(req, res) {
	res.setHeader('X-Frame-Options','SAMEORIGIN');
}

```
####明确来源白名单
Content Security Policy，简称 CSP。顾名思义，这个规范与内容安全有关，主要是用来定义页面可以加载哪些资源，减少 XSS 的发生。
```Node.js

function requestHandler(req, res) {
	res.setHeader('Content-Security-Policy',"script-src 'self'");
}

```
[Content Security Policy 介绍](https://imququ.com/post/content-security-policy-reference.html)
####禁止Content-Type嗅探
互联网上的资源有各种类型，通常浏览器会根据响应头的Content-Type字段来分辨它们的类型。例如："text/html"代表html文档，"image/png"是PNG图片，"text/css"是CSS样式文档。然而，有些资源的Content-Type是错的或者未定义。这时，某些浏览器会启用MIME-sniffing来猜测该资源的类型，解析内容并执行。
例如，我们即使给一个html文档指定Content-Type为"text/plain"，在IE8-中这个文档依然会被当做html来解析。利用浏览器的这个特性，攻击者甚至可以让原本应该解析为图片的请求被解析为JavaScript。通过下面这个响应头可以禁用浏览器的类型猜测行为：
>X-Content-Type-Options: nosniff

---
[一些安全相关的HTTP响应头](https://imququ.com/post/web-security-and-response-header.html)
