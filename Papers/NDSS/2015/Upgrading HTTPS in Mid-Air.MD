This paper has the following contributions:

1. Measurement on HSTS preload list

    - The number of entries (including different domains and base domains).

    - The Alexa rank distribution of entries.

    - Some entries are stale.
2. Measurement on HPKP preload list

    - The number of entries (including different domains and base domains).

    - What keys are in the pinned set? (CA keys or end-entity keys? How many distinct CAs do CA keys belong to?)

    - Some domains preload HPKP but don't preload HSTS.
3.  Measurement on dynamic HSTS
    - The number of domains that set an HSTS header in reponse.
    - HSTS errors of these domains including: Some domain don't deploy default HTTPS, some domain sets HSTS header only via HTTP, some domain redirect HTTPS connections (with HSTS set) to HTTP ones to a non-HSTS domain, some domain set a HSTS header that doesn't conform the grammar and some domains use a very short max-age, even 0. 

4. HSTS/HPKP and mixed content. The author think that a domain with HSTS/HPKP loads resources from a domain without HSTS/HPKP is also a mixed content case.
5. HSTS/HPKP and cookies. The author claims when a parent domain deploys HSTS/HPKP but doesn't use `includeSubDomains` directive, an attacker may steal cookies with the `domain` attribute set to the parent domain itself (if the `domain` attribute is set in `Set-Cookie` header, the cookie will be contained in the request to all subdomains of the value of the `domain` attribute) but without `secure` attribute, by intercepting an HTTP request to a subdomain.

​           

