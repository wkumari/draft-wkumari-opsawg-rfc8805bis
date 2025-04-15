---
title: "Self-Published IP Geolocation Feeds - Version 2"
abbrev: RFC8805bis
category: info

docname: draft-wkumari-opsawg-rfc8805bis-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: AREA
workgroup: WG Working Group
keyword:
  - geolocation
  - streaming
venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: wkumari/rfc8805bis
  latest: https://example.com/LATEST

author:
  -
    ins: W. Kumari
    name: Warren Kumari
    org: Google
    email: warren@kumari.net
  -
    ins: T. Pauly
    name: Tommy Pauly
    org: Apple, Inc
    email: tpauly@apple.com

normative:
  RFC7946:
  RFC8805:

informative:
  SVTA5010:
    target: "https://www.svta.org/product/svta5010/"
    title: "SVTA5010: Geo-Data for IPv6"


--- abstract

RFC8805 defines a format whereby a network operator can publish a mapping of IP
address prefixes to simplified geolocation information, colloquially termed a
"geolocation feed". Interested parties consume these feeds to update or merge
with other geolocation data sources and procedures.

This document updates RFC8805 to specify a JSON encoding for the goelocation
feed (JSON) to allow for the inclusion of additional information, and to be
more extensible.

--- middle

# Introduction

This document updates {{RFC8805}}, which defined a format whereby a network
operator can publish a mapping of IP address prefixes to simplified geolocation
information, colloquially termed a "geolocation feed". Interested parties can
poll and parse these feeds to update or merge with other geolocation data
sources and procedures.

RFC8805 specified a CSV format, which while simple, has limitations in terms of
extensibility and the amount of information that can be included. This document
specifies a JSON format for the geolocation feed to allow for the inclusion of
additional information, be more extensible, and also more easily parsed by
modern libraries.

The original CSV format in {{RFC8805}} was created in 2013, and while it has
served its purpose, many operators would like the ability to provide (and
consume) additional information associated with the geolocated prefix.

The primary additional information that is included relates to the capabilities
of the network connection associated with a prefix. For example example, cable
networks generally have high throughput and higher latency/jitter, while
cellular devices generally have smaller screens. By exposing this information
to network providers, we can help them to make better decisions about how to
optimize the content for the end user.

This is especially important for video streaming and gaming applications, where
the quality of the experience is heavily dependent on the network conditions.

The JSON format is designed to be extensible, so that new fields can be added
in the future without breaking existing implementations. This is done by using
a "key-value" pair structure, where each key is a string and the value can be
any valid JSON value (string, number, object, array, etc.). This allows for the
inclusion of additional information in the future without breaking existing
implementations. For example, if a new field is added to the JSON format,
existing implementations will simply ignore it, while new implementations can
take advantage of it.

The JSON format is also designed to be human-readable, so that operators can
easily understand the information that is being published. This is done by
using a simple and consistent naming convention for the keys, and by using a
clear and concise structure for the values.

In addition to being more extensible, experiance with RFC8805 has shown that
many prefixes share common characteristics. For example, many prefixes are
associated with a single location. In order to reduce the amount of data that
needs to be published, this document allows for the use of "prefix groups". A
prefix group is a set of prefixes that share common characteristics, and can be
represented as a single entry in the JSON format. This allows for the
publication of a single entry for a group of prefixes, rather than a separate
entry for each prefix.

{{RFC8805}} specified the location as being: alpha2code,region,city,postal_code
(with postal_code as being deprecated)

This is both simultaneously too specific and not specific enough. For example,
Tokyo covers around 2000 square kilometers, and the city of Tokyo has a
population of around 14 million people, while Thurmond, West Virginia, USA
covers less than 0.5 square kilometers and, as of 2024, had a population of 2.


To help address this, the new JSON format uses {{RFC7946}} GeoJSON format to
specify the location (with constraints to ensure that the targeting is not too
specific). This allows for the use of polygons, and other geometric shapes to
represent the location of the prefix.  ((TODO (WK): I put this in for now, but
"with constraints" is, um, handwavey. I think that this is likely already
"solved" by work in the Geoprive WG -- talk to Ray B to figure out what we
should reference, etc.!))


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Format

The new JSON format is informed both by {{SVTA5010}} and from numerous
discussions with operators, goeolocation providers, and real-time streaming
providers.

The JSON encoding of the geolocation feed is a JSON object, with the following
fields (NOTE: This is very much a first draft, and this **will** change):

  - `version`: The version of the geolocation feed format. This field is
    required, and MUST be set to "1.0".
  - `prefixes`: An array of prefix objects, each of which represents a single
    prefix in CIDR notation.
    - `connection`: The type of connection associated with the prefix. This field
      is optional, and MUST be set to one of the following values:
      - `cable`: A cable connection.
      - `fixed`: A fixed-line connection (e.g fiber, metro-Ethernet).
      - `cellular`: A cellular connection.
      - `satellite`: A satellite connection.
      - `wireless`: A wireless connection.
      - `other`: Any other type of connection. ((TODO (WK): I suspect that there
      is already a list of connection types that we can just reference. In
      addition, I suspect we should just list connection attributes, such as
      latency, bandwidth, "cost", jitter, etc. Otherwise, we are going to have to
      keep updating this document every time someone invents a new connection
      type.))
    - `location`: The location associated with the prefix. This field is
      optional, and MUST be set to a GeoJSON object. The GeoJSON object MUST be a
      `FeatureCollection` with a single `Feature` object, which MUST have a
      `geometry` field that is a `Polygon` or `MultiPolygon`.

# Security Considerations

TODO Security.

Include:

- The security considerations from {{RFC8805}}.
- The security considerations from {{RFC7946}}.
- The location specification MUST not be smaller than XX square kilometers, and
  MUST cover at least YY people. The consumer of the feed MUST not use the
  location if the location is smaller than XX. The exact values of XX and YY
  are TBD.


# IANA Considerations

This document has no IANA actions.


# Notes to self:
Also get input from:

- Ray Bellis (Geopriv chair)
- Massimo Candela (geolocate much?!)
- Wes Hardaker
- Ninrod Levy / AT&T
- Zoltán Szamonek
- Erik Kline (co-author?)
- Jason Livingood (co-author?)

ToDo:

- Add an actual schema
- Add examples
- Figure out how to do the "prefix groups" -- I think that this is just a list
  of prefixes, but we need to be careful about how we do this.
- Figure out how to specify the location:
  - I think that we should use the GeoJSON format, but we need to be careful
    about how we do this. For example, we need to make sure that the location
    is not too specific. It looks like Geopriv will be helpful here!
  - We also need to make sure that the location is not too large (e.g. a
    country).
  - We also need to make sure that the location is not too small (e.g. a single
    building).
- Figure out how to specify the connection type:
  - I suspect that there should be a registry, with some initial values.
  - We should also provide a way to specify attributes, possibly something like this:

   ```
    - `connection`: {
        "type": "cable",
        "attributes": {
          "latency": 10,
          "bandwidth": 1000,
          "jitter": 800,
          "cost": 0.01
        }
      }
      ```
  - If we list attributes, do we also want a named "type"? E.g if you know that
    the last-hop latency is 10ms, and 250Mbps, with a jitter of 800ms, do you
    care / need to know that this is "cable" vs "fiber"? Also, why would you
    care if this is "fiber" vs "ethernet" vs "free-space laser"? Ain't none of
    your business, as long as the packets get there, right?

--- back

# Acknowledgments
{:numbered="false"}

This document was inspired by discussions with numerous operators, geolocation
providers, and real-time streaming providers. In addition, the work of the
Streaming Video Technology Alliance (SVTA) in defining the SVTA5010 is
appreciated.

There have been many discussions over the years about updating this format;
unfortunately I have managed to forget who all these discussions were with. If
you are one of these people, please let me know and I will add you to the
acknowledgments!

The authors would like to thank the following people for their
input and feedback on this document: <<TODO: Add names>>.