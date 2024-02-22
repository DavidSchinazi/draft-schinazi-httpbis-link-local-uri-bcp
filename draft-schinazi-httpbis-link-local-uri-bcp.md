---
title: "Best Practices for Link-Local Connectivity in URI-Based Protocols"
abbrev: "Link-Local URI Connectivity"
category: bcp
docname: draft-schinazi-httpbis-link-local-uri-bcp-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Web and Internet Transport"
workgroup: "HTTP"
obsoletes: 6874
venue:
  group: "HTTP"
  type: "Working Group"
  mail: "ietf-http-wg@w3.org"
  arch: "https://lists.w3.org/Archives/Public/ietf-http-wg/"
  github: "DavidSchinazi/draft-schinazi-httpbis-link-local-uri-bcp"
  latest: "https://DavidSchinazi.github.io/draft-schinazi-httpbis-link-local-uri-bcp/draft-schinazi-httpbis-link-local-uri-bcp.html"
keyword:
 - IPv6 deployment
 - URI
 - URL
 - mDNS
author:
 -
    ins: D. Schinazi
    name: David Schinazi
    org: Google LLC
    street: 1600 Amphitheatre Parkway
    city: Mountain View
    region: CA
    code: 94043
    country: United States of America
    email: dschinazi.ietf@gmail.com

normative:

informative:
  URL-LS:
    title: "URL-LS"
    seriesinfo:
      WHATWG: Living Standard
    date: false
    target: "https://url.spec.whatwg.org/"
  CORS:
    title: "CORS protocol"
    seriesinfo:
      WHATWG: Living Standard
    date: false
    target: "https://fetch.spec.whatwg.org/#http-cors-protocol"
  URL-ZONE-TRACKER1:
    title: "Support IPv6 link-local addresses?"
    author:
    - org:
    date: false
    target: "https://www.w3.org/Bugs/Public/show_bug.cgi?id=27234"
  URL-ZONE-TRACKER2:
    title: Support IPv6 zone identifiers
    author:
    - org:
    date: false
    target: "https://github.com/whatwg/url/issues/392"


--- abstract

Link-local IP connectivity allows hosts on the same network to communicate with
each other without the need for centralized IP addressing infrastructure. The
IP prefixes 169.254.0.0/16 and fe80::/10 were reserved for this purpose.
However, both IP versions have limitations that make link-local addresses
unideal for usage with URI-based protocols. This document provides guidance for
how best to enable link-local connectivity in such protocols. This document
also obsoletes RFC 6874, a previous attempt at solving this issue.


--- middle

# Introduction

To simplify configuration of new hardware, manufacturers often print
configuration URLs on labels to allow the user to reach the configuration page
by copying the URL into their browser. This is often simplified further by
encoding the URL in a QR code and asking the user to scan it with a smartphone.

While the majority of IP networking uses globally routable addresses, those
rely on addressing infrastructure that isn't always available. For example, two
hosts connected via a direct peer-to-peer link may not have access to an entity
assigning IP addresses through DHCP or IPv6 router advertisements. Connectivity
is often desirable in such scenarios, and can be accomplished using link-local
addresses. This feature was added in IPv6 {{?RFC4007}} and retroactively
backported to IPv4 {{?RFC3927}}. However, these addresses have limitations that
make them unsuitable for use in URLs, as described in {{limitations}}.

This document obsoletes {{?RFC6874}}, a previous attempt at solving this
problem that failed, as described in {{attempt-6874}}. This document provides
recommendations that solve this problem in {{recommendations}}.

## Conventions and Definitions

{::boilerplate bcp14-tagged}

# Limitations

Since there is clear value in being able to configure new devices in the
absence of IP addressing infrastructure, there is interest in supporting
link-local connectivity via URLs. However, link-local addresses have
limitations in this regard.

## Limitations of Link-Local IPv4 Addresses

In order to simplify implementations (and as recommended in {{RFC3927}}), most
implementations disable IPv4 link-local addresses any time globally routeable
addresses are available. This can lead to instability of link-local addresses.
Additionally, these addresses are generated randomly with fewer than 16 bits of
entropy, making conflicts statistically likely.

Both of these limitations make it impossible for a device to print a
configuration URL on its packaging that uses a static IPv4 link-local address.

## Limitations of Link-Local IPv6 Addresses

In IPv6, link-local addresses are generated randomly with 64 bits of entropy,
making conflicts statistically unlikely. Additionally, in IPv6 the use of
multiple addresses per interface is encouraged. This allows link-local
addresses to remain even when globally routable addresses change.

However, IPv6 introduced the concept of a scope zone ({{Section 5 of RFC4007}})
and requires that every host include a zone identifier when sending to any IPv6
link-local address. While {{RFC4007}} defined a "default" zone, that is not
widely supported: most operating systems still require the scope identifier
when making a socket operation on IPv6 link-local addresses. This means that
IPv6 link-local addresses have to be accompanied by a zone identifier from the
moment that the address enters the process.

Unfortunately, IPv6 address support was added to URLs {{?RFC2732}} prior to the
creation of IPv6 scoped addresses, leaving the URL format for IPv6 addresses
incapable of representing zone identifiers. This effectively prevented the use
of link-local IPv6 addresses in URLs.

# Background

Before diving into potential solutions to these limitations, readers will
benefit from the following relevant historical context.

## URIs, URLs, and A Tale of Two Specifications

URIs and URLs were created early in the development of the World-Wide Web and
were brought to the IETF in 1994 (see {{?RFC1630}} and {{?RFC1738}}). Over the
years, the IETF maintained and evolved these specifications. In particular,
support for IPv6 addresses was added in 1999 (see {{RFC2732}}). The IETF
published an updated URI specification in 2005 ({{?RFC3986}}).

In 2004, a group of browser vendors created the WHATWG, an effort to evolve
Web-related specifications outside of the W3C or IETF. The WHATWG eventually
forked the URL specification from IETF by creating the WHATWG URL Living
Standard ({{URL-LS}}). From that point onwards, even though development of URIs
and URLs continued at the IETF, this work often didn't lead to corresponding
implementation changes in Web browsers.

Almost two decades later, the situation hasn't changed. The IETF still
maintains URL/URI specifications that are authoritative in all contexts except
the Web, while the WHATWG maintains a URL specification that is authoritative
for Web browsers. Note that the use of the word "authoritative" here is
somewhat of a misnomer: neither of these standards bodies have any actual
authority to enforce that their specifications be followed, and instead rely on
implementers choosing to follow the specification.

## IPv6 Link-Local Addresses in URLs {#attempt-6874}

As the Web gained in popularity, an increasing number of networked devices such
as routers or printers started to incorporate Web servers as their primary
means of configuration. Many of these devices function properly without
centralized IP addressing infrastructure, so there was interest in
communicating with them using IPv6 link-local addresses.

In 2004 and 2005, an effort was started to allow representing zone identifiers
in URIs {{?URI-ZONE-EARLY=I-D.fenner-literal-zone}}. That proposal leveraged
the "IPvFuture" feature of the URI specification (see {{Section 3.2.2 of
RFC3986}}), and example of the syntax is "[v1.fe80::a+en1]". That document was
never published but its format ended up being used by CUPS in 2005
({{?CUPS=I-D.sweet-uri-zoneid}}).

Over the course of 2012 and 2013, a revival of this effort led to the creation
and publication of {{RFC6874}}, an update to the IETF URL specification that
defines how to represent IP zone identifiers in URLs. This version used the
IPv6address syntax in URIs, an example of it being "[fe80::a%25en1]".

The majority of Web browsers did not implement either of these changes. The
main concern from browsers what that such a change would require modifying many
different components of the browser, with the associated security risks and
maintenance costs. Almost all browsers came to the conclusion that such a
change was not worth the effort. Further examples of what made {{RFC6874}}
complex to implement are listed in {{Section 2 of
?DRAFT-6874BIS=I-D.ietf-6man-rfc6874bis-09}}. After browsers decided not to
implement it, the WHAT URL Living Standard was updated to mark the zone
identifier as "intentionally omitted" (see {{URL-ZONE-TRACKER1}}). The WHATWG
subsequently rejected a request to add zone identifiers to their URL
specification (see {{URL-ZONE-TRACKER2}}).

## Further Attempts

After it was clear that most browser were not going to implement {{RFC6874}},
another attempt was made to modify the URI RFC: {{DRAFT-6874BIS}}. While this
attempted to address some of the difficulties in implementing {{RFC6874}}, it
did not address the fact that browsers were not willing to start such a complex
implementation effort given the small usage it was expected to receive. That
document failed to achieve consensus and was not published.

Later, an attempt was made to address the generic question of how users can
input IPv6 link-local addresses into software interfaces
{{?DRAFT-ZONE-UI=I-D.draft-carpenter-6man-zone-ui-01}}. In this model, the zone
identifier is considered independently of the IPv6 address itself. In the case
of Web browsers, the zone identifier would not be considered part of a URI.
However, this does not in itself resolve all the difficulties in considering
the zone identifier as part of the Web security model, as described in the next
section.

# Handling IPv6 Link-Local Addresses in Web Browsers

Browsers operate differently from simple command-line tools such as ping, ssh
or netcat. These tools generally take a destination as input from the
command-line, resolve that destination string into an IP address (or list of
addresses) via a function such as getaddrinfo ({{?RFC3493}}), and then
immediately perform socket operations using that address. Supporting zone
identifiers in these scenarios is pretty simple because the result of
resolution is only used in socket operations.

One might assume that Web browsers operate similarly, but that would be
incorrect. Browsers follow the Web security model, which is based around the
concept of an Origin ({{?RFC6454}}). The origin is propagated throughout the
browser software: it is involved in whether to use a resource in the HTTP cache
({{?RFC9111}}), it is checked when deciding to allow sharing information
({{CORS}}), it is used to save security policies ({{?RFC6797}}), and in many
other cases beyond making socket operations. Additionally, all the portions of
the origin are sent to the server in HTTP ({{?RFC9110}}).

In contrast, the zone identifier is only valid and meaningful in a given node.
As specified in {{RFC4007}}, the zone identifier from a given node cannot be
used by other node and it cannot be sent over the wire. This makes it
fundamentally incompatible with the concept of Origins.

The solution in {{RFC6874}} requires the browser to treat the zone identifier
as part of the origin in some contexts (such as when determining uniqueness of
a name), but not in others (such as when sending on the wire). This
significantly increases implementation complexity and security risks.

Conversely, the proposal in {{DRAFT-6874BIS}} sends the zone identifier over
the wire, and suggests that the recipient not make use of it. This creates new
implementation complexity, now on the HTTP server. It also doesn't address the
multitude of implementation changes required to incorporate the changes in URL
parsing.

In all cases, implementation matters are further complicated by the fact that
the percent character ("%") is used in URLs for percent-encoding. While each
proposal offered a different resolution to this encoding issues, all of them
have significant downsides.

Regardless of which specific mechanism is used to encode zone identifiers in
URIs, the complexity of Web browsers and widespread use of Origins make it
impossible to implement zone identifiers without large engineering efforts.

Seperately, the proposal in {{DRAFT-ZONE-UI}} would require querying the user
from many distinct browser components to determine the correct zone identifier
to use. That is particularly difficult to implement in multi-process browser
architectures. It would also confuse the user when they receive a popup for a
link-local address that appeared in HTML.

# Goal Definition {#goals}

However, the top-level goal of all these efforts is to allow link-local
communication via URLs. That does not require encoding IPv6 link-local
addresses in URIs. All that is is needed is for the URI to contain information
that resolves to the correct link-local address.

The deployment of IPv6 happened in part because it did not require users be
aware of IPv6 addresses, nor remember them, nor type them into browser address
bars. It happened transparently to the user, thanks to the DNS: once it was
possible to query IPv6 addresses from the DNS (see {{?RFC3596}}), users could
browse the Web over IPv6 without having to ever see an IPv6 address. Surfacing
IPv6 link-local addresses to users is not required.

# Recommendations for Link-Local Connectivity {#recommendations}

The concept of address resolution also applies to local connectivity in the
absence of centralized IP addressing infrastructure, because DNS hostnames can
resolve to link-local addresses. In the absence of centralized DNS
infrastructure, mDNS (see {{!RFC6762}}) can be used to resolve link-local
addresses from instance names.

Devices that are reachable over IP link-local connectivity and that host
servers of URI-based protocols SHOULD create an mDNS local instance name for
themselves and SHOULD respond to mDNS queries for that instance name.

If the instance name is defined to be unique (for example by including a unique
serial number), it can then be encoded in an URL that can be printed on the
device packaging, either as text or in the form of a QR code. Otherwise,
devices can rely on mDNS conflict resolution ({{Section 9 of RFC6762}}) to
ensure unique names, and then browse for the relevant service ({{Section 4 of
!RFC6763}}). However, Web browsers don't currently perform this browsing, so
picking a name that guarantees uniqueness is RECOMMENDED.

Following these recommendations solves the goals described in {{goals}} without
requiring any changes in Web browser software.

# Deployment Considerations

DNS Service Discovery relies on either link-local multicast (in the case of
mDNS) or on service registration infrastructure (such as
{{?DNSSD-SRP=I-D.ietf-dnssd-srp}}). If a network blocks link-local multicast
and does not offer service registration infrastructure as an alternative, then
DNS service discovery cannot function. Because of this, the recommendations in
this document won't work on such networks.

# Security Considerations

Many aspects of the Web security model rely on using a name as a root of
security. This has the following consequences:

* name unicity matters, as conflicts can lead the two devices sharing a name to
  incorrectly share a security boundary.

* HTTPS/WebPKI security currently relies on globally-registered names, and is
  therefore not available for link-local connectivity. Such link-local
  communication is therefore inherently not authenticated. Future work might
  define mechanisms to trust local anchors, which would enable such security.

Fundamentally, mDNS has similar security properties as the underlying
link-local address it resolves to.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

Some of the historical context in this document came from prior research
documented in {{?URL-HISTORY=I-D.ruby-url-problem}}. The author would like to
thank {{{Brian E. Carpenter}}}, {{{Stuart Cheshire}}}, and {{{Bob Hinden}}} for
their prior work in this space. Additionally, the author thanks {{{Eric
Rescorla}}} and {{{Michael Sweet}}} for their review and comments.
