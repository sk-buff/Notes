### Main Goals

This paper is the prototype of HSTS. It tries to use ForceHTTPS to modify a browser's behavior when accessing a website that has deployed ForceHTTPS as following:

(1) HTTP connections are upgraded to HTTPS ones, which can prevent HTTP interception. Nowadays, almost all articles that talk about HSTS regard this as the most important security mechanism introduced by HSTS.

(2) All TLS errors (mainly refers to certificate validation error I think) are treated as fatal which means users can not ignore these errors and continue to access the website. This property ensures security in such a scenario: an active attacker (the paper refers attackers in (1) as passive attackers) can use a self-signed certificate to intercept the HTTPS traffic, although users will see the certificate validation error in their browsers, some of them will ignore it and try to continue to visit the website (this behavior is called **click through**), so the cookies set by the original website even with the **secure** directive will be stole by the attacker. Although rarely mentioned, this is one important property of ForceHTTPS (or HSTS), it's also implemented in Chrome (I didn't test it in Firefox).

(3) Mix content is forbidden. This property is not part of HSTS standard today, because browsers have implemented this separately.

### Shortcomings 

**·** This paper uses cookies to send FroceHTTPS info to browsers instead of an HTTP header. This leads to a problem that a subdomain may set an FroceHTTPS cookie to influence it's parent domain (e.g. break the accessibility of the parent domain). 

**·** This paper also discusses some inherent problems of storing states in user's browser, e.g., how to organize the local storage and the privacy problem. But as I said, there problems are inherent, I don't think there are good ways to solve them. 