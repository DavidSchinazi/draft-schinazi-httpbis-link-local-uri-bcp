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
how best to enable such link-local connectivity in such protocols. This
document also obsoletes RFC 6874, a previous attempt at solving this issue.


--- middle

# Introduction

While the majority of IP networking uses globally routable addresses, those
rely on addressing infrastructure that isn't always available. For example, two
hosts connected via a direct peer-to-peer link may not have access to an entity
assigning IP addresses through DHCP or IPv6 router advertisements. Connectivity
is often desirable in such scenarios, and can be accomplished using link-local
addresses. This feature was added in IPv6 {{?RFC4007}} and retroactively
backported to IPv4 {{?RFC3927}}. However, these addresses have limitations:

* In IPv4, a given interface on a given host can generally only have one
  address. Therefore, most implementations will disable IPv4 link-local
  addresses any time globally routeable addresses are available. Additionally,
  these addresses are generated randomly with fewer than 16 bits of entropy,
  making conflicts statistically likely.

* In IPv6, link-local addresses are generated randomly with 64 bits of entropy,
  making conflicts statistically unlikely. Additionally, in IPv6 the use of
  multiple addresses per interface is encouraged. This allows link-local
  addresses to remain even when globally routable addresses change. However,
  IPv6 introduced the concept of a scope zone ({{Section 5 of RFC4007}}) and
  requires that every host include a zone identifier when sending to any IPv6
  link-local address. Unfortunately, IPv6 address support was added to URLs
  {{?RFC2732}} prior to the creation of IPv6 scoped addresses, which
  effectively prevented the use of link-local IPv6 addresses in URLs.

This document obsoletes {{?RFC6874}}, a previous attempt at solving this
problem that failed, as described in {{attempt-6874}}. This document provides
recommendations for another solution to this problem in {{recommendations}}.

## Conventions and Definitions

{::boilerplate bcp14-tagged}

# Background

## URIs, URLs, and A Tale of Two Specifications

URIs and URLs were created early in the development of the World-Wide Web and
were brought to the IETF in 1994 (see {{?RFC1630}} and {{?RFC1738}}). Over the
years, the IETF maintained and evolved these specifications. In particular,
support for IPv6 addresses was added in 1999 (see {{RFC2732}}). The IETF
published an updated URI specification in 2005 ({{?RFC3986}}).

In 2004, a group of browser vendors created the WHATWG, an effort to evolve
Web-related specifications outside of the W3C or IETF. The WHATWG initially
forked the HTML specification from the W3C, and later also forked the URL
specification from IETF by creating the WHATWG URL Living Standard
({{URL-LS}}). From that point onwards, even though development of URIs and URLs
continued at IETF, this work no longer had any impact on Web browsers.

Almost two decades later, the situation hasn't changed. The IETF still
maintains URL/URI specifications that are authoritative in all contexts except
the Web, while the WHATWG maintains a URL specification that is authoritative
for Web browsers. Note that the use of the word authoritative here is somewhat
of a stretch: neither of these standards bodies have any actual authority to
enforce that their specifications be followed, and instead rely on implementers
choosing to follow the specification.

## IPv6 Link-Local Addresses in URLs {#attempt-6874}

As the Web gained in popularity, and increasing number of networked devices
such as routers or printers started to incorporate Web servers as their primary
means of communication. Many of these devices function properly without
centralized IP addressing infrastructure, so there was interest in
communicating with them using IPv6 link-local addresses.

Over the course of 2012 and 2013, this led to the publication of {{RFC6874}},
an update to the IETF URL specification that defines how to represent IP zone
identifiers in URLs. This didn't lead to any implementation changes in Web
browsers. The main concern from browsers what that such a change would require
modifying many different components of the browser, with the associated
security risks and maintenance costs. All browsers independently came to the
same conclusion that such a change was not worth the effort. Further examples
of what made {{RFC6874}} complex to implement are listed in {{Section 2 of
?RFC6874BIS=I-D.ietf-6man-rfc6874bis-09}}. After browsers decided not to
implement it, the WHAT URL Living Standard was updated to mark the zone
identifier as "intentionally omitted" (see {{URL-ZONE-TRACKER1}}). The WHATWG
subsequently rejected a request to add zone identifiers to their URL
specification (see {{URL-ZONE-TRACKER2}}).

## Further Attempts

After it was clear no browser was going to implement {{RFC6874}}, another
attempt was made to modify the URI RFC: {{RFC6874BIS}}. While this attempted to
address some of the difficulties in implementing {{RFC6874}}, it did not
address the fact that browsers were not willing to start such a complex
implementation effort given the small usage it was expected to receive. That
document failed to achieve consensus and was never published.

Later, another attempt was made to allow Web browsers to communicate via IPv6
link-local addresses: {{?ZONE-UI=I-D.draft-carpenter-6man-zone-ui-01}}. In this
attempt, the zone identifier is no longer encoded in the URI. Instead client
applications are requested to offer UI to allow selecting the zone identifier.
While that document does not mention the Web or browsers directly, the
intention was to eventually leverage it to help convince browsers to implement
support for IPv6 link-local addresses. Similarly, this proposal does not seem
to be gaining support from browser vendors.

# Recommendations for Link-Local Connectivity {#recommendations}

Instead of focusing on IPv6 link-local addresses, it is important to remember
that the goal is to enable link-local connectivity, not necessarily to surface
IPv6 link-local addresses to users. The deployment of IPv6 happened in part
because it did not require users be aware of IPv6 addresses, not remember them,
not type them into browser address bars. It happened transparently to the user,
thanks to the DNS: once it was possible to query IPv6 addresses from the DNS
(see {{?RFC3596}}), users could browse the web over IPv6 without having to ever
see an IPv6 address.

This logic also applies to local connectivity in the absence of centralized IP
addressing infrastructure, because DNS hostnames can resolve to link-local
addresses. In the absence of centralized DNS infrastructure, mDNS (see
{{!RFC6762}}) can be used to resolve link-local addresses from instance names.

Devices that are reachable over IP link-local connectivity and that host
servers of URI-based protocols SHOULD create an mDNS local instance name for
themselves and SHOULD respond to mDNS queries for that instance name.

# Security Considerations

Many aspects of the Web security model rely on using a name as a root of
security. This has the following consequences:

* name unicity matters, as conflicts can lead the two devices sharing a name to
  incorrectly share a security boundary.

* HTTPS/WebPKI security relies on globally-registered names, and is therefore
  not available for link-local connectivity. Such link-local communication is
  therefore inherently not authenticated.

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
their prior work in this space.
