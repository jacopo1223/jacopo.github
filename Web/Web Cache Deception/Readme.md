# Web Cache Deception
The vulnerability found concerns a misconfiguration of the web cache that allows an attacker to access private or sensitive content intended only for authenticated users.
By manipulating the URL, the attacker can trick the cache system and force the caching of a response containing confidential information.

# How to exploit
Here is a concrete example of how a discrepancy in path mapping within a web app can be exploited to perform a Web Cache Deception attack. Suppose we have a web app with an area reserved for user profiles

The authenticated user can access their profile page via the following URL:
**GET /user/123/profile HTTP/1.1**

**Host: www.example.com**

Server response:

HTTP/1.1 200 OK
Content-Type: text/html
Cache-Control: no-store
Cf-Cache-Status: BYPASS

<html>
<body>
Profilo di Jacopo
<p>Email: jacopo@example.com</p>
<p>Indirizzo: xxxxx</p>
</body>
</html>

In this case, the server correctly returns the profile information of authenticated user 123 (jacopo) and does not cache the response thanks to the Cache-Control: no-store header

**Attempted Web Cache Deception:**

The attacker could manipulate the URL by adding a commonly cacheable extension such as .js, thereby tricking the cache system into storing the response

**GET /user/123/profile.js HTTP/1.1
Host: www.example.com**

Server response:

HTTP/1.1 200 OK
Content-Type: text/html
Cache-Control: public, max-age=3600
Cf-Cache-Status: **HIT**

 the origin server might treat the URL **/user/123/profile.js** as if it were a normal request for user profile 123, ignoring the .js extension. However, the caching system might interpret the URL as a static resource (a .js file) and decide to cache the response


At this point, any user (authenticated or not) accessing the same manipulated URL in incognito mode or from another device will get the response cached with the sensitive profile data of user 123

**GET /user/123/profile.js HTTP/1.1
Host: www.example.com**

# Recommendations:
Disable caching for sensitive content: Correctly set cache headers, such as Cache-Control: no-store, for all sensitive resources and pages requiring authentication.

Path validation: Implement control logic on URLs to reject requests with unexpected extensions or unusual syntax.

Authentication control: Ensure that every request to sensitive resources always performs an authentication check, even if the resource has been previously cached.





