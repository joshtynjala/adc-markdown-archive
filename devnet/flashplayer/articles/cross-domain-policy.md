# Cross-domain policy file usage recommendations for Flash Player

by Lucas Adamski

![Lucas Adamski](./img/1296458201149.jpg)

## Requirements

### User level

Intermediate

When an attempt is made to load content into a SWF file at runtime, the request
is subject to the Flash Player security model, which is in place to protect
users and website owners. As part of this model, Flash Player by default
prevents cross-domain loading of data, but allows cross-domain sending of data.

This security model was set up to parallel the default settings provided in most
web browsers. Flash Player does, however, allow you to make exceptions by
placing a cross-domain policy file on the server where the content is stored.
Cross-domain policy files are a Flash Player security control that you can use
to enable data loading between domains. This powerful functionality allows
Flash- and Flex-based rich Internet applications (RIAs) to exchange information
in ways that are not possible in applications built with AJAX, DHTML, or
JavaScript.

This article discusses some of the common security issues that you should
consider when deciding how to use a cross-domain policy file on your server. In
general, websites using cross-domain policy files increase their security
exposure. This is because the cross-domain policy file used by Flash Player
allows access to information by more domains than are allowed in the default
configuration. As with any security mechanism, use of the cross-domain policy
requires careful analysis of the proposed application architecture and threat
model to understand potential risks.

**Note:** Using a cross-domain policy file could expose your site to various
attacks. Please read this document before hosting a cross-domain policy.

### Cookie authentication

Sites that use the common practice of authentication based on cookies to access
private or user-specific data should be especially careful when using
cross-domain policy files. Currently, the major web browsers include
domain-specific cookies on most web requests. Even when cross-domain requests
are made in Flash Player, browsers include the cookies of the domain that handle
the request. In other words, a SWF file that is hosted on one server can make a
request to any other server, and the browser automatically appends the cookies
to that request.

If a website uses cookies only for authentication, then enabling the
cross-domain policy file allows authenticated requests to be made from one
domain against the other domain at any time when the browser has valid cookies.
Depending on how authorization is restricted on the website, this could
inadvertently expose data to other domains or allow invocation of functionality
across domains. The cross-domain policy file should permit only domains that can
be trusted to make requests that include the user's domain-specific cookies.
(You should allow access only for servers in other domains that you control,
have a contract or partnership with, or in some other way you feel won't cause
you problems.)

As an example, a user is logged in to an e-commerce site that uses cookies for
authentication. On the site is a user account settings page where you can see
information such as your mailing address and other personally identifiable
information. If this site has an overly permissive cross-domain policy file like
`*`, a SWF file that is hosted on another domain could silently load the account
settings data and send it elsewhere. This is because the browser appends the
cookies for the e-commerce site to the request from Flash Player.

It has become quite common for an application to include a set of public XML web
services or SOAP APIs to use in mashups. The website hosting that application
may also have user-specific content that is authenticated using cookies.
Cross-domain policy files are often used to allow access to these public APIs.
It is in those cases where special care must be taken to ensure that allowing
cross-domain access to public APIs does not inadvertently expose functionality
that is intended only to be used by the browser-based UI.

One approach to limiting exposure of cookies to cross-domain calls is to place
XML web services or SOAP APIs that will be made accessible through a
cross-domain policy file onto a separate domain or virtual host. This
architecture provides the most visible and easily understood boundary between
services. Flash Player can load data only from an exact-match domain, so by
placing the APIs and the cross-domain file on a separate domain, you leave the
authenticated area to the default policy.

Of course, using separate domains will not be possible for all sites. Any site
that is considering deployment of a cross-domain policy file should also
consider exposing only those virtual directories that are intended to be shared.
This can be accomplished by placing cross-domain policy files into each of the
virtual directories that should be exposed.

Cross-domain policy files in specific directories indicate that the policy
applies only to the contents of that virtual directory and below. For many
environments, a single virtual directory that includes all public methods may be
the most appropriate architecture. It is important to note that any cross-domain
policy files that are not on the root level of the domain must be explicitly
loaded by the Flash movie.

### Network topology

If trust boundaries are defined by network topology (for instance, if a data
source is available only on an intranet), then those trust boundaries should be
reflected within any cross-domain policy.

By default, the Flash Player security model reinforces trust based on network
topology because only SWFs provided by a specific domain are able to make data
loading requests against that domain. But any cross-domain policy that is more
permissive than the default should consider whether any topologic features
should be incorporated into the cross-domain policy. Placing an overly
permissive cross-domain policy file onto a server that depends entirely on
topology to establish trust boundaries could add risk to the application.

In the modern corporate intranet, for example, many web applications are
provided that are considered confidential to the company but are available to
all internal employees. Because the corporation has a firewall and the websites
are only internally routable, application-level authentication is often not
implemented. Instead, the natural boundary established by the network topology
is the only authentication mechanism. But client-side scripts such as JavaScript
or ActionScript are run on the local computer rather than the server from where
the code originated, so they often disrupt trust assumptions based on topology.

For example, a website located at app.internal.foo.com might want to allow any
other internal.foo.com website to connect to the site. This can be accomplished
by providing a policy file that allows `*.internal.foo.com`. If that policy file
were to be expanded to allow all domains (`*`), then SWFs hosted on any website
on the Internet would be able to access the website, if they are accessed by a
client within the internal network.

### Cross-site request forgery (XSRF)

The attachment of cookies to all requests is a frequently misunderstood property
of the browser that has led to a category of web application vulnerabilities
known as
[cross-site request forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery)
(XSRF). An XSRF is not unique to Flash-based applications or the cross-domain
policy file, but some solutions to mitigating XSRF may be affected by the policy
file.

An XSRF vulnerability can occur any time a web application has sensitive
functionality that can be exercised in a single atomic request. A cross-domain
request takes advantage of the cookies included with the request to initiate
requests that are authenticated but may be malicious.

XSRF has affected web applications for as long as they have used cookies to
implement authentication. Recently, XSRF vulnerabilities have received much
attention within the security community because they have been discovered in
some widely used web applications.

At present, most browser-based technologies can make cross-domain data sending
requests (but not data loading requests). From HTML and JavaScript, data sending
requests include loading frame content, form posts, images, etc. From Flash
Player, data sending request can be used to display images, play video and
sound, etc. All of these data sending methods are unable to directly interact
with the information returned after the request is made. At present, most
browser-based technologies do not allow cross-domain requests that retrieve
data, such as the Ajax `XMLHttpRequest` or Flash Player `XML.load` method.

Common techniques for reducing exposure to XSRF include limiting the duration of
cookies, checking the referrer field, and adding non-cookie based authentication
information such as a time-based hash message authentication code (HMAC) or
state machine in a hidden field. Because cross-domain policy files allow
inspection of the data loaded from another site, they can limit the
effectiveness of some in-band non-cookie authentication mechanisms such as
hidden fields in document state management. Carefully review any application for
XSRF and determine whether the countermeasures will be sufficient before making
changes to a site's cross-domain policy.

### Recommendations

This article recommends a number of specific security issues that should be
reviewed prior to deployment of a cross-domain policy file:

- Carefully evaluate which sites will be allowed to make cross-domain calls.
  Consider network topology and any authentication mechanisms that will be
  affected by the configuration or implementation of the cross-domain policy.
- Limit the scope of the cross-domain policy to only the desired functionality
  by creating subdomains or virtual directories containing shared functionality.
- Review any XSRF prevention mechanisms to see if they may be affected by
  allowing cross-domain data loading.

### Where to go from here

For more information on the cross-domain policy file and Flash Player security
in general, please see the following resources:

- [Adobe Flash Player 9 security white paper ](https://web.archive.org/web/20140406220927/http://wwwimages.adobe.com/www.adobe.com/content/dam/Adobe/en/devnet/flashplayer/pdfs/flash_player_9_security.pdf)
  (PDF, 588K)
