---
title: >
  Ordered maps and multimaps in CBOR and CDDL
abbrev: New maps in CBOR and CDDL
docname: draft-bormann-cbor-cddl-ordered-multi-maps-latest
date: 2021-04-01

stand_alone: true

ipr: trust200902
keyword: Internet-Draft
cat: info

pi: [toc, sortrefs, symrefs, compact, comments]

author:
  -
    name: Carsten Bormann
    org: Universität Bremen TZI
    street: Postfach 330440
    city: Bremen
    code: D-28359
    country: Germany
    phone: +49-421-218-63921
    email: cabo@tzi.org
    role: editor
  -
       name: Brendan Moran
       organization: Arm Limited
       email: Brendan.Moran@arm.com
  -
       name: Henk Birkholz
       organization: Fraunhofer SIT
       email: henk.birkholz@sit.fraunhofer.de
  - name: Emile Cormier
    email: emile.cormier.jr@gmail.com

contributor:
 - name: Kio Smallwood
 - name: Joe Hildebrand

normative:
  RFC8610: cddl
  RFC8949: cbor
  IANA.cddl: control
  IANA.cbor-tags: tags
# informative:


--- abstract

The Concise Binary Object Representation (CBOR, RFC 8949) is a data
format whose design goals include the possibility of extremely small
code size, fairly small message size, and extensibility without the
need for version negotiation.

Basic CBOR supports unordered maps free of duplicate keys, similar to the
way JSON defines JSON objects (RFC 8259).
Using the CBOR extension point of tags, tags for a selection of variants of
maps and multimaps have been registered, but gaps remain.
The present document defines a consolidated set of CBOR tags for
map-like data items involving key-value pairs.

The Concise Data Definition Language (CDDL), standardized in RFC 8610,
is often used to express CBOR data structure specifications.
It provides "control operators" as its main language extension point.

The present document defines a number of control operators that enable
the description of CBOR data structures that make use of the newly
defined tags or that employ the same underlying structures.


--- middle

# Introduction        {#intro}

(See abstract for now.)

Terminology
-----------

{::boilerplate bcp14}

This specification uses terminology from {{-cbor}} and {{-cddl}}.
In particular, with respect to CDDL control operators, "target" refers to
the left hand side operand, and "controller" to the right hand side operand.
The terms "array" and "map" (if unadorned) refer to CBOR major type 4
and CBOR major type 5; this is not called out explicitly.


# CBOR tags for map-like data items

This document defines a consolidated set of CBOR tags for map-like
entities involving key-value pairs. These tags encode the following
meta-data concerning map-like data items:

- the homogeneity of the key and value types,
- whether the order of the key-value pairs carries semantic value
  ("ordered") or needs to be ignored ("unordered"),
- the uniqueness of the keys, and,
- the major type used to encode the key-value pairs.

Note that the term "ordered" as used in this document is distinct from
"sorted" -- "ordered" implies that the order in the data item
interchanged conveys a semantically relevant ordering, while a
property "sorted" can easily be established after interchange (by,
simply, sorting), less often needs to be indicated, and is more
complex to indicate as it may need details about the sorting

## Summary

| Tag | LSBs | Homogeneous Value | Homogeneous Key | Ordering  | Duplicate Keys Allowed | Data Item | Related Tag |
| --- | ---- | -------------     | -----------     | --------- | ---------------------- | --------- | ----------- |
| 128 | 0000 | No                | No              | Unordered | No                     | map       | 259         |
| 129 | 0001 | No                | No              | Unordered | Yes                    | array     | TDB280*     |
| 130 | 0010 | No                | No              | Ordered   | No                     | array     | TBD279*     |
| 131 | 0011 | No                | No              | Ordered   | Yes                    | array     |             |
| 132 | 0100 | No                | Yes             | Unordered | No                     | map       | 275         |
| 133 | 0101 | No                | Yes             | Unordered | Yes                    | array     |             |
| 134 | 0110 | No                | Yes             | Ordered   | No                     | array     |             |
| 135 | 0111 | No                | Yes             | Ordered   | Yes                    | array     |             |
| 136 | 1000 | Yes               | Yes             | Unordered | No                     | map       |             |
| 137 | 1001 | Yes               | Yes             | Unordered | Yes                    | array     |             |
| 138 | 1010 | Yes               | Yes             | Ordered   | No                     | array     |             |
| 139 | 1011 | Yes               | Yes             | Ordered   | Yes                    | array     |             |
{: #tag-new title="New CBOR tags defined in this document"}

*TBD279: https://github.com/Sekenre/cbor-ordered-map-spec/blob/master/CBOR_Ordered_Map.md

*TBD280: https://github.com/ecorm/cbor-tag-multimap

\[The intention of the present document is to obviate the need for defining TBD279/TBD280.]

{{proglan}} provides information about constructs in a few programming
languages that are related to the tags being defined.

## Key/Value Type

Bits 2/3 of the tag provides information on the map's key and value types:

- 0b00xx Unspecified: There is no specified type for the map's keys and values
- 0b01xx Homogeneous Key: All keys have the same data type
- 0b10xx Homogeneous Key/Value: All values have the same data type in addition to all keys having the same data type (the types for keys and values may be distinct).

The semantics for homogeneity shall be the same as for RFC8746 homogeneous arrays (tag 41). That is, "which CBOR data items constitute elements of the same application type is specific to the application" (RFC8746, subsection 3.2).

Maps with arbitrary keys and homogeneous values are considered unusual, so they are left out of this specification in order to reduce the number of tags allocated (12 instead of 16).

## Ordering

Bit 1 of the tag represents the map's ordering semantics:

- 0: The order of key-value pairs is unspecified
- 1: Key-value pairs are encoded in the same order in which they were inserted

## Key Uniqueness

Bit 0 of the tag represents the uniqueness of the map's keys.

- 0: Keys are unique
- 1: Keys may be duplicated (i.e. multimaps)

## Data Item

All these map-like data items could be represented as a tag with an
enclosed array of alternating key-value pairs, as
in:

~~~CBORdiag
129(["key1", 1, "key2", 2])
~~~

However, representing the key-value pairs as a CBOR map
for those cases where this is possible enables generic decoders that
are oblivious of these tags to represent the data in a more
appropriate platform type.

Specifically, the key-value pairs are represented as a map if and only if

- the ordering is unspecified and
- the keys are unique;

they are represented as an array of alternating keys
and values otherwise.

## Related Tags (Informative)

### Tag 259

Specification: https://github.com/shanewholloway/js-cbor-codec/blob/master/docs/CBOR-259-spec--explicit-maps.md

The above defined tag 128 may be used instead to guide a JavaScript decoder into interpreting a CBOR map as a JavaScript Map instead of an Object.

### Tag 275

Specification: https://github.com/ecorm/cbor-tag-text-key-map

The above defined tag 132 may be used instead to guide a decoder into interpreting a CBOR map as a JavaScript-like Object having only text string keys. The decoder would have to verify the first key to establish that the map has homogeneous text string keys.

### Tag TBD279 ###

Draft specification: https://github.com/Sekenre/cbor-ordered-map-spec/blob/master/CBOR_Ordered_Map.md

The above defined tag 130 may be used instead to encode map-like data items where the
order of the key-value pairs is semantically significant.

### Tag TDB280 ###

Draft specification: https://github.com/ecorm/cbor-tag-multimap

The above defined tag 129 may be used instead to encode a multimap as an array of key-value pairs.


CDDL Support for Map-Like Data Items
============

The Concise Data Definition Language (CDDL), standardized in RFC 8610,
provides "control operators" as its main language extension point.

The present document defines a number of control operators that enable
the use of group notation (enclosed in a CDDL map) to specify any of
the above map-like data structures:

| Name    | Purpose                |
| .omm    | Ordered (Multi-)Map    |
| .umm    | Unordered (Multi-)Map  |
| .unique | Uniqueness requirement |
{: #tbl-new title="New control operators in this document"}

## Map notation for map-like data items

\[needs better examples]

CDDL already can describe both arrays of alternating keys and values
and maps (unordered and with unique keys).  The two control operators
`.omm` and `.umm`
introduced in this section enable the use of CDDL map notation for
map-like types beyond actual maps, increasing readability and possibly
even reusability.

In a simple example that provides an unordered collection of zero or more
home addresses and zero or more work addresses, each labeled as such,
we use traditional map notation to describe that collection:

~~~cddl
[* (text, any)] .umm {
  * home: address
  * work: address
  $$more-addresses
}
~~~

The `.omm` and `.umm` control operators convert a group definition
enclosed into a CDDL map given as a controller type into an array type
given as the target type.
The controller type given is unwrapped ({{Section 3.7 of RFC8610}}) into a
group.  Keys and values of the entries in that group are then alternatingly
matched as elements in the target array.
Note that both target and controller type can contribute to the
shaping of the data; declaring the key type as `text` limits what can
be added to the `$$more-addresses` socket.

`.omm` and `.umm` differ in the semantics of the array type created:
`.omm` defines an ordered (multi)map, i.e., the order of the key/value
element pairs in the array matters, while `.umm` defines an unordered
(multi)map, i.e., data items that present the same set of key/value
pairs in different orders are equivalent.

Note that the ability to specify specific ("homogeneous") types is
provided by the ability to specify the target type, as in the example above.

Note that there is not strictly a need to define a control operator for building
unordered maps with non-duplicate keys, as existing CBOR maps already
fill this role, however the use of a map type as the target is allowed for symmetry
(implying uniqueness of the keys), allowing the following:

~~~cddl
{* text => any} .umm {
  ? home: address
  ? work: address
  $$more-addresses
}
~~~

## Uniqueness

The `.unique` control annotates the target as requiring uniqueness,
within the enclosing container(*), of its value, among the other data
items in that enclosing container that are also marked `.unique`, with the
same controller.

E.g.,

~~~cddl
feature-set = [* feature .unique "set"]
ordered-pairs-with-unique-keys-and-values =
  [* (any .unique "key", any .unique "value") ]
~~~

defines a `feature-set` as an array of zero or more `feature` values
that need to be all different (as they are unique in `set`), and
`ordered-map-with-unique-keys-and-values` as an array of zero pairs of
keys and values, where the keys need to be unique among themselves and
the values need to be unique among themselves (the latter example
could employ an `.omm` or `.umm` operator to further restrict what can
be in these keys and values).

> Discussion: (*) while it is probably not a big problem to define
> what exactly the "enclosing" container is, it may be useful to
> actually define a larger scope of the uniqueness.  CDDL currently
> does not have a way to establish and point to such a larger scope;
> we might define one ad hoc here or leave that for later extension.


CDDL typenames
==========

For the use with CDDL {{-cddl}}, the typenames defined in {{tag-cddl}}
are recommended unless there is a need for more specific shaping of
the data.

~~~ CDDL
anymap = {* any => any}

tbd128 = #6.128(anymap)
tbd129 = #6.129([* (any, any)] .umm anymap)
tbd130 = #6.130([* ((any .unique "mm"), any)] .omm anymap)
tbd131 = #6.131([* (any, any)] .omm anymap)
tbd132<k> = #6.132({* k => any})
tbd133<k> = #6.133([* (k, any)] .umm anymap)
tbd134<k> = #6.134([* ((k .unique "mm"), any)] .omm anymap)
tbd135<k> = #6.135([* (k, any)] .omm anymap)
tbd136<k,v> = #6.136({* k => v})
tbd137<k,v> = #6.137([* (k, v)] .umm anymap)
tbd139<k,v> = #6.138([* ((k .unique "mm"), v)] .omm anymap)
tbd139<k,v> = #6.139([* (k, v)] .omm anymap)
~~~
{: #tag-cddl title="Recommended typenames for CDDL"}

\[fill in better names for tbdnnn]

Note that there is no need to call out the uniqueness of the keys
explicitly in tbd128, tbd132, or tbd136, as the use of maps as a
representation format already provides that key uniqueness.

IANA Considerations
==================

## Tags


IANA is requested to allocate the tags of {{tag-new}} in the CBOR tags
registry {{-tags}}, using this document as the specification
reference.

The allocations are requested to be assigned from the "specification
required" space (24..255).


## CDDL control operators

This document requests IANA to register the contents of
{{tbl-iana-reqs}} into the CDDL Control Operators registry {{-control}}:

| Name    | Reference |
| .omm    | [RFCthis] |
| .umm    | [RFCthis] |
| .unique | [RFCthis] |
{: #tbl-iana-reqs title=="New control operators to be registered"}

Implementation Status
=====================

TBD

Security considerations
=======================

The security considerations of {{-cddl}} apply.

--- back

# Implementation Considerations

This non-normative appendix provides information about the use on
implementations of the tags and control operators defined.

## Programming Language Containers (Informative) {#proglan}

The following subsections describe how the tags in this document relate to various programming language containers. Containers that are not part of the programming language or its standard libraries are not considered here.

The *Encoding Tag* column in the following tables provide the recommended tag that best represents the given container type. For example, it's possible to
use tag 132 for encoding an ECMAScript `Map` if all keys happen to be of the same type, however tag 128 is more general and applies to any `Map`. When encoding
an ECMAScript `Object`, tag 128 would be technically correct but is too general; tag 132 best presents the fact that an `Object` has text keys only.

The *Decodable Tags* column in the following tables, are for data items can be decoded into the destination container without having to inspect the following:

- the uniqueness of the keys,
- the ordering of the keys, and,
- the data types of **every** keys/value pair.

It may however be necessary to inspect the data types of the **first** key-value pair in the case of tags representing homogeneous keys/values.

### ECMAScript

Container         | Encoding Tag | Decodable Tags |
----------------- | ------------ | -------------- |
`Object`          | 132          | 132, 136       |
`Map`             | 128          | 128, 132, 136  |
`Array` of pairs  | 131          | All            |

### Python

Container           | Encoding Tag | Decodable Tags |
------------------- | ------------ | -------------- |
`TypedDict`         | 136          | 136            |
`namedtuple`        | 132          | 132, 136       |
`dict`              | 128          | 128, 132, 136  |
`OrderedDict`       | 130          | 130, 134, 138  |
`list` of 2-tuples  | 131          | All            |

### C++

Container(s)           | Encoding Tag | Decodable Tags |
---------------------- | ------------ | -------------- |
`Map<K, T>`            | 136          | 136            |
`Map<K, D>`            | 132          | 132, 136       |
`Map<D, D>`            | 128          | 128, 132, 136  |
`MultiMap<K, T>`       | 137          | 137            |
`MultiMap<K, D>`       | 133          | 133            |
`MultiMap<D, D>`       | 129          | 128, 129       |
`Sequence<Pair<K, T>>` | 139          | \[136, 139\]   |
`Sequence<Pair<K, D>>` | 135          | \[132, 139\]   |
`Sequence<Pair<D, D>>` | 131          | All            |

Legend:

- `K`: Static key type
- `T`: Static value type
- `D`: Suitable dynamic type, such as `std::any` or `std::variant`
- `Map`: `std::map` or `std::unordered_map`
- `MultiMap`: `std::multimap` or `std::unordered_multimap`
- `Sequence`: Sequence container that maintains order (e.g. `std::vector`)
- `Pair`: Object containing a key and a value, such as `std::pair`, or `std::tuple`.

Note that a C++ `std::map` stores its key-value pairs in a sorted fashion, and
does **not** preserve insertion order in the same manner as Python's `OrderedDict`.

## CDDL Implementation Considerations

TBD

Acknowledgements
================
{: numbered="no"}

The CBOR tags defined in this document were developed by Emile Cormier under the sponsorship of Duc Luong,
based on discussions with Kio Smallwood and Joe Hildebrand.
The CDDL control operators defined in this document were developed by
Carsten Bormann, Brendan Moran, and Henk Birkholz.

<!--  LocalWords:  dedenting dedented
 -->
