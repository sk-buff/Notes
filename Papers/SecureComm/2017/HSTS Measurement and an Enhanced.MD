This paper discusses some flaws in HSTS deployment, which are:

1\. Send HSTS with HTTP header.

2\. Lexical errors in HSTS header.

3\. HSTS max-age too long or too short.

4\. Redirection problem, includes:

- Some HTTP requests are redirects to HTTPS requests but there's no HSTS header in the HTTPS request, but HSTS exists in the response of a direct HTTPS request.

- When a mobile browser accesses `*http://www.example.com*`, the request will finally be redirected to `https://m.example.com`, the most secure direction chain is firstly redirects `http://www.example.com` to `https://www.example.com` and set the HSTS header in the latter response, then redirects `https://www.example.com` to `https://m.example.com`. But some domains don't such a redirection chain.

5\. Users may input an URL without scheme when adding a new bookmark. In this case when users click this bookmark the browser will send an HTTP request. (**I think this one is meaningless.**)

6\. An interesting problem is that the author claims they can get an HTTPS response with HSTS when accessing `www.baidu.com` in China, but when they access the same site in America, HSTS header is absent.

This paper claims the old SSL striping attack no longer works because the promotion of dynamic websites (which means there may be some HTTPS requests sent to the server while users are browsing the original intercepted web page) and some client side SSL striping detection mechanisms (e.g. read `location.protocol` in a script). So it proposes a new SSL striping attack which (1) adds a `submit` DOM-3 event listener to strip all HTTPS requests to HTTP ones by injecting a script. (2) finds and modifies the SSL striping detection script (this can not be achieved by directly writing `location.protocol` because it will cause the redirection of the page).