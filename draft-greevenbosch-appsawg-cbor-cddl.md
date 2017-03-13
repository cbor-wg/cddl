---
stand_alone: true
ipr: trust200902
docname: draft-greevenbosch-appsawg-cbor-cddl-latest
cat: info
pi:
  toc: 'yes'
  symrefs: 'yes'
  compact: 'yes'
  comments: 'yes'
title: >
  CBOR data definition language (CDDL):
  a notational convention to express CBOR data structures
abbrev: CDDL
area: Applications
wg: ''
date: 2017-03-13
author:
- ins: H. Birkholz
  name: Henk Birkholz
  org: Fraunhofer SIT
  abbrev: Fraunhofer SIT
  email: henk.birkholz@sit.fraunhofer.de
  street: Rheinstrasse 75
  code: '64295'
  city: Darmstadt
  country: Germany
- ins: C. Vigano
  name: Christoph Vigano
  org: Universität Bremen
  abbrev: Universität Bremen
  email: christoph.vigano@uni-bremen.de
- ins: C. Bormann
  name: Carsten Bormann
  org: Universitaet Bremen TZI
  street:
  - Bibliothekstr. 1
  city: Bremen
  code: D-28359
  country: Germany
  phone: +49-421-218-63921
  email: cabo@tzi.org
normative:
  RFC2119:
  RFC3629:
  RFC5234:
  RFC7049:
  RFC7159:
informative:
  RELAXNG:
    title: RELAX-NG Compact Syntax
    target: http://relaxng.org/compact-20021121.html
    author:
      organization: OASIS
    date: 2002-11-21
  RFC7071:
  I-D.newton-json-content-rules:

--- abstract


This document proposes a notational convention to express CBOR data
structures (RFC 7049).
Its main goal is to provide an easy and unambiguous way to express structures
for protocol messages and data formats that use CBOR.

--- middle

# Introduction

In this document,
a notational convention to express CBOR {{RFC7049}} data structures is defined.

The main goal for the convention is to provide a unified notation that can
be used when defining protocols that use CBOR.  We term the convention
"CBOR data definition language", or CDDL.

The CBOR notational convention has the following goals:

{: style="format (G%d)"}
* Provide an unambiguous description of the overall structure of a CBOR data structure.

* Flexibility to express the freedoms of choice in the CBOR data format.

* Possibility to restrict format choices where appropriate [^_format].

* Able to express common CBOR datatypes and structures.

* Human and machine readable and processable.

* Automatic checking of data format compliance.

* Extraction of specific elements from CBOR data for further processing.

This document has the following structure:

The syntax of CDDL is defined in {{syntax}}.
Examples of CDDL and related CBOR data instances are defined in
{{examples}}. {{usage}} discusses usage of CDDL. Examples are provided
early in the text to better illustrate concept definitions.
A formal definition of CDDL using ABNF grammar is provided in {{abnf}}.
Finally, a prelude of standard CDDL definitions available in every
CBOR specification is listed in {{prelude}}.

[^_format]: So far, the ability to restrict format choices have not
    been needed beyond the floating point formats.  Those can be
    applied to ranges using the new .and annotation now.  It is not
    clear we want to add more format control before we have a use case.

## Requirements notation

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in RFC
2119, BCP 14 {{RFC2119}}.

## Terminology

New terms are introduced in _cursive_.
CDDL text in the running text is in `typewriter`.

# The Style of Data Structure Specification

CDDL focuses on styles of specification that are in use in the
community employing the data model as pioneered by JSON and now
refined in CBOR.

There are a number of more or less atomic elements of a CBOR data
model, such as numbers, simple values (false, true, nil), text and
byte strings;
CDDL does not focus on specifying their structure.  CDDL of course
also allows adding a CBOR tag to a data item.

[^_reptype]: We don't have a way yet to qualify the representation of a
      value, e.g., whether it is float16, float32 or float64.  TO DO:
      probably borrowing something from diagnostic notation (section 6.1 RFC
      7049).

The more important components of a data structure definition language
are the data types used for composition: arrays and maps in CBOR
(called arrays and objects in JSON).  While these are only two
representation formats, they are used to specify four loosely
distinguishable styles of composition:

* A _vector_, an array of elements that are mostly of the same
  semantics.  The set of signatures associated with a signed data item
  is a typical application of a vector. <!-- [^_example1] -->
* A _record_, an array the elements of which have different,
  positionally defined semantics, as detailed in the data structure
  definition.  A 2D point, specified as an array of an x coordinate
  (which comes first) and a y coordinate (coming second) is an example
  of a record, as is the pair of exponent (first) and mantissa
  (second) in a CBOR decimal fraction. <!-- [^_example1] -->
* A _table_, a map from a domain of map keys to a domain of map
  values, that are mostly of the same semantics.  A set of language
  tags, each mapped to a text string translated to that specific language,
  is an example of a table.  The key domain is usually not limited to
  a specific set by the specification, but open for the
  application, e.g., in a table mapping IP addresses to MAC addresses,
  the specification does not attempt to foresee all possible IP addresses. <!-- [^_example1] -->
* A _struct_, a map from a domain of map keys as defined by the
  specification to a domain of map values the semantics of each of
  which is bound to a specific map key.  This is what many people have
  in mind when they think about JSON objects; CBOR adds the ability to
  use map keys that are not just text strings.  Structs can be used to
  solve similar problems as records; the use of explicit map keys
  facilitates optionality and extensibility. <!-- [^_example1] -->

[^_example1]: If there are suitable examples later on, maybe link them
    here already? Maybe not.

Two important concepts provide the foundation for CDDL:

1. Instead of defining all four types of composition in CDDL separately,
or even defining one kind for arrays (vectors and records) and one
kind for maps (tables and structs), there is only one kind of
composition in CDDL: the _group_ ({{group}}).

2. The other important concept is that of a _type_.  The entire CDDL
specification defines a type (the one defined by its first _rule_),
which formally is the set of CBOR instances that are acceptable for
this specification.  CDDL predefines a number of basic types such as
`uint` (unsigned integer) or `tstr` (text string), often making use of
a simple formal notation for CBOR data items.  Each value that
can be expressed as a CBOR data item also is a type in its own right,
e.g. `1`.  A type can be built as a _choice_ of other types, e.g., an
`int` is either a `uint` or a `nint` (negative integer).
Finally, a type can be built as an array or a map from a group.

## Groups and Composition in CDDL {#group}

CDDL Groups are lists of name/value pairs (group _entries_).

In an array context, only the value of the entry is represented; the name is
annotation only (and can be left off if not needed).
In a map context, the names become the map keys ("member keys").

In an array context, the sequence of elements in the group is
important, as it is the information that allows associating actual
array elements with entries in the group.
In a map context, the sequence of entries in a group is not relevant
(but there is still a need to write down group entries in a sequence).

A group can be placed in (round) parentheses, and given a name by
using it in a rule:

{:cddl: artwork-align="center"}


~~~~ CDDLx
pii = (
  age: int,
  name: tstr,
  employer: tstr,
)
~~~~
{:cddl #basic-group title="A basic group"}


Or a group can just be used in the definition of something else:

~~~~ CDDL
person = {(
  age: int,
  name: tstr,
  employer: tstr,
)}
~~~~
{:cddl #group-in-map title="Using a group in a map"}

which, given the above rule for pii, is identical to:

~~~~ CDDLx
person = {
  pii
}
~~~~
{:cddl #group-by-name title="Using a group by name"}

Note that the (curly) braces signify the creation of a map; the groups
themselves are neutral as to whether they will be used in a map or an array.

The parentheses for groups are optional when there is some other set
of brackets present, so it would be slightly more
natural to express {{group-in-map}} as:

~~~~ CDDL
person = {
  age: int,
  name: tstr,
  employer: tstr,
}
~~~~
{:cddl}

Groups can be used to factor out common parts of structs, e.g.,
instead of writing:

~~~~ CDDL
person = {
  age: int,
  name: tstr,
  employer: tstr,
}

dog = {
  age: int,
  name: tstr,
  leash-length: float,
}
~~~~
{:cddl}

one can choose a name for the common subgroup and write:

~~~~ CDDL
person = {
  identity,
  employer: tstr,
}

dog = {
  identity,
  leash-length: float,
}

identity = (
  age: int,
  name: tstr,
)
~~~~
{:cddl #group_factorization title="Using a group for factorization"}

Note that the contents of the braces in the above definitions
constitute (anonymous) groups, while `identity` is a named group.

### Usage {#composition_usage}

Groups are the instrument used in composing data structures with
CDDL. It is a matter of style in defining those structures whether to
define groups (anonymously) right in their contexts or whether to
define them in a separate rule and to reference them with their
respective name (possibly more than once).

With this, one is allowed to define all small parts of their data structures
and compose bigger protocol units with those or to have only one big
protocol data unit that has all definitions ad hoc where needed.

<!-- Duuh, we need a better way to point to RELAXNG.
If it is obvious that a group is defined ad hoc, the round brackets can be
omitted, allowing for an even more concise notation and a certain familiarity
with the syntax of other notational languages such as JSON or {{RELAXNG}}. [^_EXAMPLE2]

[^_EXAMPLE2]: This could certainly benefit from an example?
 -->

### Syntax {#composition_syntax}

The composition syntax intends to be concise and easy to read:

* The start of a group can be marked by '('
* The end of a group can be marked by ')'
* Definitions of entries inside of a group are noted as follows:
*keytype => valuetype,* (read "keytype maps to valuetype").
The comma is actually optional (not just in the final entry), but it is
considered good style to set it.  The double arrow can be replaced by a colon
in the common case of directly using a text string as a key (see {{structs}}).

An entry consists of a *keytype* and a *valuetype*:

* *keytype* is either an atom used as the actual key or a type in general.
  The latter case may be needed when using groups in a table context, where the actual
  keys are of lesser importance than the key types, e.g in contexts verifying
  incoming data.
* *valuetype* is a type, which could be derived from the major types
  defined in {{RFC7049}}, could be a convenience valuetype defined in this
  document ({{prelude}}) or the name of a type defined in the specification.

A group definition can also contain choices between groups, see {{choices}}.

## Types

### Values

Values such as numbers and strings can be used in place of a type.
(For instance, this is a very common thing to do for a keytype, common
enough that CDDL provides additional convenience syntax for this.)

### Choices

Many places that allow a type also allow a choice between types,
delimited by a "/" (slash).  The entire choice construct can be put
into parentheses if this is required to make the construction
unambiguous (please see {{abnf}} for the details).

Choices of values can be used to express enumerations:

~~~~ CDDL
attire = "bow tie" / "necktie" / "Internet attire"
protocol = 6 / 17
~~~~
{:cddl}

Similarly as for types, CDDL also allows choices between groups,
delimited by a "//" (double slash).

~~~~ CDDL
address = { delivery }

delivery = (
street: tstr, ? number: uint, city //
po-box: uint, city //
per-pickup: true )

city = (
name: tstr, zip-code: uint
)
~~~~
{:cddl}

Both for type choices and for group choices, additional alternatives
can be added to a rule later in separate rules by using "/=" and "//=",
respectively, instead of "=":

~~~~
attire /= "swimwear"

delivery //= (
lat: float, long: float, drone-type: tstr
)
~~~~
{:cddl}

It is not an error if a name is first used with a "/=" or "//="
(there is no need to "create it" with "=").

#### Ranges

Instead of naming all the values that make up a choice, CDDL allows
building a _range_ out of two values that are in an ordering
relationship.
A range can be inclusive of both ends given (denoted by joining two
values by ".."), or include the first and exclude the second (denoted
by instead using "...").

[^_range]:  TO DO: define this precisely.  This clearly includes integers and
        floats.  Strings -- as in "a".."z" -- could be added if desired, but
        this would require adopting a definition of string ordering and
        possibly a successor function so "a".."z" does not include "bb".

~~~~ CDDL
device-address = byte
max-byte = 255
byte = 0..max-byte ; inclusive range
first-non-byte = 256
byte1 = 0...first-non-byte ; byte1 is equivalent to byte
~~~~
{:cddl}

CDDL currently only allows ranges between numbers [^_range].

#### Turning a group into a choice

Some choices are built out of large numbers of values, often integers,
each of which is best given a semantic name in the specification.
Instead of naming each of these integers and then accumulating these
into a choice, CDDL allows building a choice from a group by prefixing
it with a "&" character:

<!-- color = [terminal-color, extended-color] -->
~~~~ CDDL
terminal-color = &basecolors
basecolors = (
  black: 0, red: 1,  green: 2,  yellow: 3,
  blue: 4,  magenta: 5,  cyan: 6,  white: 7,
)
extended-color = &(
  basecolors,
  orange: 8,  pink: 9,  purple: 10,  brown: 11,
)
~~~~
{:cddl}

As with the use of groups in arrays ({{arrays}}), the membernames have
only documentary value (in particular, they might be used by a tool
when displaying integers that are taken from that choice).

### Representation Types

CDDL allows the specification of a data item type by referring to the
CBOR representation (major and minor numbers).  How this is used
should be evident from the prelude ({{prelude}}).

It may be necessary to make use of representation types outside the
prelude, e.g., a specification could start by making use of an
existing tag in a more specific way, or define a new tag not defined
in the prelude:

~~~~ CDDL
my_breakfast = #6.55799(breakfast)   ; cbor-any is too general!
breakfast = cereal / porridge
cereal = #6.998(tstr)
porridge = #6.999([liquid, solid])
liquid = milk / water
milk = 0
water = 1
solid = tstr
~~~~
{:cddl}

### Root type

There is no special syntax to identify the root of a CDDL data
structure definition: that role is simply taken by the first rule
defined in the file.

This is motivated by the usual top-down approach for defining data
structures, decomposing a big data structure unit into smaller parts;
however, except for the root type, there is no need to strictly follow
this sequence.

(Note that there is no way to use a group as a root -- it must be a
type.
Using a group as the root might be employed as a way to specify a CBOR
sequence in a future version of this specification; this would act as
if that group is used in an array and the data items in that fictional
array form the members of the CBOR sequence.)

# Syntax {#syntax}

In this section, the overall syntax of CDDL is shown, alongside some
examples just illustrating syntax.
(The definition will not attempt to be overly formal; refer to {{abnf}}
for the details.)

## General conventions

The basic syntax is inspired by ABNF {{RFC5234}}, with

* rules, whether they define groups or types, are defined with a name,
  followed by an equals sign "=" and the actual definition according
  to the respective syntactic rules of that definition.

*   A name can consist of any of the characters from the set
    {'A', ..., 'Z', 'a', ..., 'z', '0', ..., '9', '\_', '-', '@', '.',
    '$'}, starting with
    an alphabetic character (including '@', '\_', '$') and ending in one
    or a digit.
    *     Names are case sensitive.
    *     It is preferred style to start a name with a lower case letter.
    *     The hyphen is preferred over the underscore (except in a
          "bareword" ({{structs}}), where the semantics may actually require an underscore).
    *     The period may be useful for larger specifications, to express
          some module structure (as in `tcp.throughput`
          vs. `udp.throughput`).
    *     A number of names are predefined in the CDDL prelude, as listed in {{prelude}}.
    *     Rule names (types or groups) do not appear in the actual CBOR encoding, but
          names used as "barewords" in member keys do.

*   Comments are started by a ';' (semicolon) character and finish at the end of a line
(LF or CRLF).

*   outside strings, whitespace (spaces, newlines, and comments) is used
    to separate syntactic elements for readability (and to separate
    identifiers or numbers that follow each other); it is otherwise
    completely optional.

*   Hexadecimal numbers are preceded by '0x' (without quotes, lower case
    x), and are case insensitive.  Similarly, binary numbers are preceded
    by '0b'. <!-- ABNF borken here -->

*   Text strings are enclosed by double quotation '"' characters.
They follow the conventions for strings as defined in section 7 of {{RFC7159}}.
(ABNF users may want to note that there is no support in CDDL for the
concept of case insensitivity in text strings; if necessary, regular
expressions can be used ({{regexp}}).)

*    Byte strings are enclosed by single quotation "'" characters and may
be prefixed by "h" or "b64".  If unprefixed, the string is interpreted
as with a text string, except that single quotes must be escaped and
that the UTF-8 bytes resulting are marked as a byte string (major type
2).
If prefixed as "h" or "b64", the string is interpreted as a sequence
of hex digits or a base64(url) string, respectively (as with the
diagnostic notation in section 6 of {{RFC7049}}; cf. {{textbin}}); any white space
present within the string (including comments) is ignored in the prefixed case.
[^_strings]


[^_strings]: TO DO: This still needs to be fully realized in the ABNF and in the CDDL tool.


*   CDDL uses UTF-8 {{RFC3629}} for its encoding.

Example:

~~~~ CDDL
; This is a comment
person = { g }

g = (
  "name": tstr,
  age: int,  ; "age" is a bareword
)
~~~~
{:cddl}

## Occurrence {#occurrence}

An optional _occurrence_ indicator can be given in front of a
group entry.  It is either one of the characters '?' (optional), '\*'
(zero or more), or '+' (one or more), or is of the form n\*m, where n
and m are optional unsigned integers and n is the lower limit (default
0) and m is the upper limit (default no limit) of occurrences.

If no occurrence indicator is specified, the group entry is to occur exactly
once (as if 1\*1 were specified).

Note that CDDL, outside any directives/annotations that could possibly
be defined, does not make any
prescription as to whether arrays or maps use the definite length or
indefinite length encoding.  I.e., there is no correlation between
leaving the size of an array "open" in the spec and the fact that it
is then interchanged with definite or indefinite length.

Please also note that CDDL can describe flexibility that the data
model of the target representation does not have.  This is rather
obvious for JSON, but also is relevant for CBOR:

~~~ CDDL
apartment = {
  kitchen: size,
  * bedroom: size,
}
size = float ; in m2
~~~

The previous specification does not mean that CBOR is changed to allow
to use the key "bedroom" more than once.  In other words, due to the
restrictions imposed by the data model, the third line pretty much turns
into:

~~~ CDDLx
  ? bedroom: size,
~~~

(Occurrence indicators beyond one still are useful in maps for groups
that allow a variety of keys.)

## Predefined names for types

CDDL predefines a number of names.
This subsection summarizes these names, but please see {{prelude}} for
the exact definitions.

The following keywords for primitive datatypes are defined:

"bool"
:   Boolean value (major type 7, additional information 20 or 21).

"uint"
:   An unsigned integer (major type 0).

"nint"
:   A negative integer (major type 1).

"int"
:   An unsigned integer or a negative integer.

"float16"
:   IEEE 754 half-precision float (major type 7, additional information 25).

"float32"
:   IEEE 754 single-precision float (major type 7, additional information 26).

"float64"
:   IEEE 754 double-precision float (major type 7, additional information 27).

"float"
:   One of float16, float32, or float64.

"bstr" or "bytes"
:   A byte string (major type 2).

"tstr" or "text"
:   Text string (major type 3)

(Note that there are no predefined names for arrays or maps; these are
defined with the syntax given below.)

In addition, a number of types are defined in the prelude that are
associated with CBOR tags, such as `tdate`, `bigint`, `regexp` etc.


## Arrays

Array definitions surround a group with square brackets.

For each entry, an occurrence indicator as specified in {{occurrence}} is permitted.

For example:

~~~~ CDDL
unlimited-people = [* person]
one-or-two-people = [1*2 person]
at-least-two-people = [2* person]
person = (
    name: tstr,
    age: uint,
)
~~~~
{:cddl}

The group `person` is defined in such a way that repeating it in the
array each time generates alternating names and ages, so these are four valid
values for a data item of type `unlimited-people`:

~~~~ CBORdiag
["roundlet", 1047, "psychurgy", 2204, "extrarhythmical", 2231]
[]
["aluminize", 212, "climograph", 4124]
["penintime", 1513, "endocarditis", 4084, "impermeator", 1669,
 "coextension", 865]
~~~~
{:cddl}

## Maps

The syntax for specifying maps merits special attention, as well as a
number of optimizations and conveniences, as it is likely to be the
focal point of many specifications employing CDDL.  While the syntax
does not strictly distinguish struct and table usage of maps, it
caters specifically to each of them.

### Structs

The "struct" usage of maps is similar to the way JSON objects are used
in many JSON applications.

A map is defined in the same way as defining an array (see {{arrays}}), except
for using curly braces "{}" instead of square brackets "[]".

An occurrence indicator as specified in {{occurrence}} is permitted for each
group entry.

The following is an example of a structure:

~~~~ CDDL
Geography = [
  city           : tstr,
  gpsCoordinates : GpsCoordinates,
]

GpsCoordinates = {
  longitude      : uint,            ; multiplied by 10^7
  latitude       : uint,            ; multiplied by 10^7
}
~~~~
{:cddl}

When encoding, the Geography structure is encoded using a CBOR array
with two entries (the keys for the group entries are ignored),
whereas the GpsCoordinates are encoded as a CBOR map with two
key-value pairs.

Types used in a structure can be defined in separate rules or just in
place (potentially placed inside parentheses, such as for choices).
E.g.:

~~~~ CDDL
located-samples = {
  sample-point: int,
  samples: [+ float],
}

~~~~
{:cddl}

where `located-samples` is the datatype to be used when referring to the struct,
and `sample-point` and `samples` are the keys to be used.
This is actually a complete example: an identifier that is followed by a
colon can be directly used as the text string for a member key (we speak of a
"bareword" member key), as can a double-quoted string or a number.
(When other types, in particular multi-valued ones, are used as
keytypes, they are followed by a double arrow, see below.)

If a text string key does not match the syntax for an identifier (or
if the specifier just happens to prefer using double quotes), the text
string syntax can also be used in the member key position, followed by
a colon.  The above example could therefore have been written with
quoted strings in the member key positions.

All the types defined can be used in a keytype position by following them with a
double arrow.  A string also is a (single-valued) type, so another
form for this example is:

~~~~ CDDL
located-samples = {
  "sample-point" => int,
  "samples" => [+ float],
}
~~~~
{:cddl}

A better way to demonstrate the double-arrow use may be:

~~~~ CDDL
located-samples = {
  sample-point: int,
  samples: [+ float],
  * equipment-type => equipment-tolerances,
}
equipment-type = [name: tstr, manufacturer: tstr]
equipment-tolerances = [+ [float, float]]
~~~~
{:cddl}

The example below defines a struct with optional entries: display name
(as a text string), the name components first name and family name (as a
map of text strings), and age information (as an unsigned integer).

~~~~ CDDL
PersonalData = {
  ? displayName: tstr,
  NameComponents,
  ? age: uint,
}

NameComponents = (
  ? firstName: tstr,
  ? familyName: tstr,
)
~~~~
{:cddl}

Note that the group definition for NameComponents does not generate
another map; instead, all four keys are directly in the struct built
by PersonalData.

In this example, all key/value pairs are optional from the perspective of CDDL.
With no occurrence indicator, an entry is mandatory.

If the addition of more entries not specified by the current
specification is desired, one can add this possibility explicitly:

~~~~ CDDL
PersonalData = {
  ? displayName: tstr,
  NameComponents,
  ? age: uint,
  * tstr => any
}

NameComponents = (
  ? firstName: tstr,
  ? familyName: tstr,
)
~~~~
{:cddl #xmp-personaldata title="Personal Data: Example for extensibility"}

The cddl tool ({{tool}}) generated as one acceptable instance for this specification:

~~~~ CBORdiag
{"familyName": "agust", "antiforeignism": "pretzel",
 "springbuck": "illuminatingly", "exuviae": "ephemeris",
 "kilometrage": "frogfish"}
~~~~
{:cddl}

(See {{socketplug}} for one way to explicitly identify an extension point.)

### Tables

A table can be specified by defining a map with entries where the
keytype is not single-valued, e.g.:

~~~~ CDDL
square-roots = {* x => y}
x = int
y = float
~~~~
{:cddl}

Here, the key in each key/value pair has datatype x (defined as int),
and the value has datatype y (defined as float).

If the specification does not need to restrict one of x or y (i.e., the application is free to
choose per entry), it can be replaced by the predefined name `any`.

As another example,
the following could be used as a conversion table converting from an integer
or float to a string:

~~~~ CDDL
tostring = {* mynumber => tstr}
mynumber = int / float
~~~~
{:cddl}


## Tags {#tagsec}

A type can make use of a CBOR tag (major type 6) by using the
representation type notation, giving #6.nnn(type) where nnn is an
unsigned integer giving the tag number and `type` is the type of the
data item being tagged.

For example, the following line from the CDDL prelude ({{prelude}}) defines `biguint` as
a type name for a positive bignum N:

~~~~ CDDL
biguint = #6.2(bstr)
~~~~
{:cddl}

The tags defined by {{RFC7049}} are included in the prelude.
Additional tags since registered need to be added to a CDDL
specification as needed; e.g., a binary UUID tag could be referenced
as `buuid` in a specification after defining

~~~~ CDDL
buuid = #6.37(bstr)
~~~~
{:cddl}

In the following example, usage of the tag 32 for URIs is optional:

~~~~ CDDL
my_uri = #6.32(tstr) / tstr
~~~~
{:cddl}


## Annotations

An _annotation_ allows to annotate a _target_ type with a _control_
type via an _annotator_.

The syntax for an annotated type is `target .annotator control`, where annotators are
special identifiers prefixed by a dot.  (Note that _target_ or _control_
might need to be parenthesized.)

Three annotators are defined at his point.  Note that the CDDL tool
does not currently support combining multiple annotations on a single
target.

### Annotation .size

A `.size` annotation controls the size of the target in bytes by the
control type.  Examples:

~~~~ CDDL
full-address = [[+ label], ip4, ip6]
ip4 = bstr .size 4
ip6 = bstr .size 16
label = bstr .size (1..63)
~~~~
{:cddl #annotate-size title="Annotation for size in bytes"}

When applied to an unsigned integer, the `.size` annotation restricts
the range of that integer by giving a maximum number of bytes that
should be needed in a computer representation of that unsigned integer.
In other words, `uint .size N` is equivalent to `0...BYTES_N`, where
BYTES_N == 256**N.

~~~~ CDDL
audio_sample = uint .size 3 ; 24-bit, equivalent to 0..16777215
~~~~
{:cddl #annotate-int-size title="Annotation for integer size in bytes"}

Note that, as with value restrictions in CDDL, this annotation is not
a representation constraint; a number that fits into fewer bytes can
still be represented in that form, and an inefficient implementation
could use a longer form (unless that is restricted by some format
constraints outside of CDDL, such as the rules in Section 3.9 of
{{RFC7049}}).

### Annotation .bits

A `.bits` annotation on a byte string indicates that, in the
target, only the bits numbered by a number in the control type are
allowed to be set.  (Bits are counted the usual way, bit number `n`
being set in `str` meaning that `(str[n >> 3] & (1 << (n & 7))) != 0`.)[^_bitsendian]

Similarly, a `.bits` annotation on an unsigned integer `i` indicates
that for all unsigned integers `n` where `(i & (1 << n)) != 0`, `n`
must be in the control type.

~~~~ CDDL
tcpflagbytes = bstr .bits flags
flags = &(
  fin: 8,
  syn: 9,
  rst: 10,
  psh: 11,
  ack: 12,
  urg: 13,
  ece: 14,
  cwr: 15,
  ns: 0,
) / (4..7) ; data offset bits

rwxbits = uint .bits rwx
rwx = &(r: 2, w: 1, x: 0)
~~~~
{:cddl #annotate-bits title="Annotation for what bits can be set"}

The CDDL tool generates the following ten example instances for `tcpflagbytes`:

~~~~ CBORdiags
h'906d' h'01fc' h'8145' h'01b7' h'013d' h'409f' h'018e' h'c05f'
h'01fa' h'01fe'
~~~~
{:cddl}

These examples do not illustrate that the above CDDL specification
does not explicitly specify a size of two bytes: A valid all clear
instance of flag bytes could be `h''` or `h'00'` or even `h'000000'`
as well.

[^_bitsendian]: How useful would it be to have another variant that
        counts bits like in RFC box notation?  (Or at least per-byte?
        32-bit words don't always perfectly mesh with byte strings.)

### Annotation .regexp {#regexp}

A `.regexp` annotation indicates that the text string given as a
target needs to match the PCRE regular expression given as a value in the
control type, where that regular expression is anchored on both sides.
(If anchoring is not desired for a side, `.*` needs to be inserted there.)

~~~~ CDDL
nai = tstr .regexp "\\w+@\\w+(\\.\\w+)+"
~~~~
{:cddl #annotate-regexp title="Annotation with a PCRE regexp"}

The CDDL tool proposes:

~~~~ CBORdiag
"N1@CH57HF.4Znqe0.dYJRN.igjf"
~~~~
{:cddl}


### Annotations .cbor and .cborseq

A `.cbor` annotation on a byte string indicates that
the byte string carries a CBOR encoded data item.  Decoded, the
data item matches the type given as the right-hand side argument
(type1 in the following example).

> `bytes .cbor type1`



Similarly, a `.cborseq` annotation on a byte string indicates that
the byte string carries a sequence of CBOR encoded data items.
When the data items are taken as an array, the array matches the type
given as the right-hand side argument (type2 in the following example).

> `bytes .cborseq type2`

(The conversion of the encoded sequence to an array can be effected
for instance by wrapping the byte string between the two bytes 0x9f
and 0xff and decoding the wrapped byte string as a CBOR encoded data
item.)

### Annotations .within and .and

A `.and` annotation on a type indicates that the data item matches
both that left hand side type and the type given as the right hand side.
(Formally, the resulting type is the intersection of the two types
given.)

> `type1 .and type2`

A variant of the `.and` annotation is the `.within` annotation, which
expresses an additional intent: the left hand side type is meant to be
a subset of the right-hand-side type.

> `type1 .within type2`

While both forms have the identical formal semantics (intersection),
the intention of the `.within` form is that the right hand side gives
guidance to the types allowed on the left hand side, which typically
is a socket ({{socketplug}}):

~~~~ CDDL
message = $message .within message-structure
message-structure = [message_type, *message_option]
message_type = 0..255
message_option = any

$message /= [3, dough: text, topping: [* text]]
$message /= [4, noodles: text, sauce: text, parmesan: bool]
~~~~

For `.within`, a tool might flag an error if type1 allows data items
that are not allowed by type2.  In contrast, for `.and`, there is no
expectation that type1 already is a subset of type2.

### Annotations .lt, .le, .gt, .ge, .eq, .ne, and .default

The annotations .lt, .le, .gt, .ge, .eq, .ne specify a constraint on
the left hand side type to be a value less than, less than or equal,
equal to, not equal to, greather than, or greater than or equal to a
value given as a (single-valued) right hand side type.
In the present specification, the first four annotations (.lt, .le,
.gt, .ge) are defined only for numeric types, as these have a natural
ordering relationship.

~~~~ CDDL
speed = number .ge 0  ; unit: m/s
~~~~

A variant of the `.ne` annotation is the `.default` annotation, which
expresses an additional intent: the value specified by the
right-hand-side type is intended as a default value for the left hand
side type given, and the implied .ne annotation is there to prevent
this value from being sent over the wire.
This annotation is only meaningful when the annotated type is used in
an optional context; otherwise there would be no way to express the
default value.

~~~~ CDDL
timer = {
  time: uint,
  ? displayed-step: (number .gt 0) .default 1
}
~~~~

## Socket/Plug

Both for type choices and group choices, a mechanism is defined that
facilitates starting out with empty choices and assembling them later,
potentially in separate files that are concatenated to build the full
specification.

Per convention, CDDL extension points are marked with a leading dollar
sign (types) or two leading dollar signs (groups).  Tools honor that
convention by not raising an error if such a type or group is not
defined at all; the symbol is then taken to be an empty type choice
(group choice), i.e., no choice is available.

~~~~ CDDL
tcp-header = {seq: uint, ack: uint, * $$tcp-option}

; later, in a different file

$$tcp-option //= (
sack: [+(left: uint, right: uint)]
)

; and, maybe in another file

$$tcp-option //= (
sack-permitted: true
)
~~~~
{:cddl}

Names that start with a single "$" are "type sockets", names with a
double "$$" are "group sockets".  It is not an error if there is no
definition for a socket at all; this then means there is no way to
satisfy the rule (i.e., the choice is empty).

All definitions (plugs) for socket names must be augmentations, i.e., they
must be using "/=" and "//=", respectively.

To pick up the example illustrated in {{xmp-personaldata}}, the
socket/plug mechanism could be used as shown in
{{xmp-personaldata-sp}}:


~~~~ CDDL
PersonalData = {
  ? displayName: tstr,
  NameComponents,
  ? age: uint,
  * $$personaldata-extensions
}

NameComponents = (
  ? firstName: tstr,
  ? familyName: tstr,
)

; The above already works as is.
; But then, we can add later:

$$personaldata-extensions //= (
  favorite-salsa: tstr,
)

; and again, somewhere else:

$$personaldata-extensions //= (
  shoesize: uint,
)
~~~~
{:cddl #xmp-personaldata-sp title="Personal Data example: Using socket/plug extensibility"}


## Operator Precedence

As with any language that has multiple syntactic features such as
prefix and infix operators, CDDL has operators that bind more tightly
than others.  This is becoming more complicated than, say, in ABNF, as
CDDL has both types and groups, with operators that are specific to
these concepts.  Type operators (such as "/" for type choice) operate on
types, while group operators (such as "//" for group choice) operate
on groups.  Types can simply be used in groups, but groups need to be
bracketed (as arrays or maps) to become types.  So, type operators
naturally bind closer than group operators.

For instance, in

~~~
t = [group1]
group1 = (a / b // c / d)
a = 1 b = 2 c = 3 d = 4
~~~

group1 is a group choice between the type choice of a and b and the
type choice of c and d.  This becomes more relevant once member keys
and/or occurrences are added in:

~~~
t = {group2}
group2 = (? ab: a / b // cd: c / d)
a = 1 b = 2 c = 3 d = 4
~~~

is a group choice between the optional member "ab" of type a or b and
the member "cd" of type c or d.  Note that the optionality is attached
to the first choice ("ab"), not to the second choice.

Similarly, in

~~~
t = [group3]
group3 = (+ a / b / c)
a = 1 b = 2 c = 3
~~~

group3 is a repetition of a type choice between a, b, and c[^unflex];  if just
a is to be repeatable, a group choice is needed to focus the occurrence:

[^unflex]: A comment has been that this is counter-intuitive. One solution
    would be to simply disallow unparenthesized usage of occurrence indicators
    in front of type choices unless a member key is also present like in group2 above.

~~~
t = [group4]
group4 = (+ a // b / c)
a = 1 b = 2 c = 3
~~~

group4 is a group choice between a repeatable a and a single b or c.

In general, as with many other languages with operator precedence
rules, it is best not to rely on them, but to insert parentheses for
readability:

~~~
t = [group4a]
group4a = ((+ a) // (b / c))
a = 1 b = 2 c = 3
~~~

The operator precedences, in sequence of loose to tight binding, are
defined in {{abnf}} and summarized in {{prec}}.  (Arities given are 1
for unary prefix operators and 2 for binary infix operators.)

| Operator | Ar | Operates on               | Prec |
| =        |  2 | name = type, name = group |    1 |
| /=       |  2 | name /= type              |    1 |
| //=      |  2 | name //= group            |    1 |
| //       |  2 | group // group            |    2 |
| ,        |  2 | group, group              |    3 |
| *        |  1 | \* group                  |    4 |
| N*M      |  1 | N*M group                 |    4 |
| +        |  1 | + group                   |    4 |
| ?        |  1 | ? group                   |    4 |
| \=>      |  2 | type => type              |    5 |
| :        |  2 | name: type                |    5 |
| /        |  2 | type / type               |    6 |
| &        |  1 | &group                    |    6 |
| ..       |  2 | type..type                |    7 |
| ...      |  2 | type...type               |    7 |
| .anno    |  2 | type .anno type           |    7 |
{: #prec cols="c r l r" title="Summary of operator precedences"}

# Examples {#examples}

This section contains various examples of structures defined using CDDL.

## Fruit

{{exCBORstruct}} contains an example for a CBOR structure that contains information about
fruit.

~~~~ CDDL
fruitlist = [* Fruit]

Fruit = {
  name                 : tstr,
  colour               : [* color],
  avg_weight           : float16,
  price                : uint,
  international_names  : International,
  rfu                  : bstr,           ; reserved for future use
}

International = {
  "DE"                 : tstr,           ; German
  "EN"                 : tstr,           ; English
  "FR"                 : tstr,           ; French
  "NL"                 : tstr,           ; Dutch
  "ZH-HANS"            : tstr,           ; Chinese
}

color = &(
  black: 0, red: 1,  green: 2,  yellow: 3,
  blue: 4,  magenta: 5,  cyan: 6,  white: 7,
)
~~~~
{:cddl}
{: #exCBORstruct title='Example CBOR structure' artwork-align="center"}

## RFC 7071 {#example-7071}

{{RFC7071}} defines the Reputon structure for JSON using somewhat
formalized English text. Here is a (somewhat verbose) equivalent
definition using the same terms, but notated in CDDL:

~~~~ CDDL
reputation-object = {
  reputation-context,
  reputon-list
}

reputation-context = (
  application: text
)

reputon-list = (
  reputons: reputon-array
)

reputon-array = [* reputon]

reputon = {
  rater-value,
  assertion-value,
  rated-value,
  rating-value,
  ? conf-value,
  ? normal-value,
  ? sample-value,
  ? gen-value,
  ? expire-value,
  * ext-value,
}

rater-value = ( rater: text )
assertion-value = ( assertion: text )
rated-value = ( rated: text )
rating-value = ( rating: float16 )
conf-value = ( confidence: float16 )
normal-value = ( normal-rating: float16 )
sample-value = ( sample-size: uint )
gen-value = ( generated: uint )
expire-value = ( expires: uint )
ext-value = ( text => any )
~~~~
{:cddl}

An equivalent, more compact form of this example would be:

~~~~ CDDL
reputation-object = {
  application: text
  reputons: [* reputon]
}

reputon = {
  rater: text
  assertion: text
  rated: text
  rating: float16
  ? confidence: float16
  ? normal-rating: float16
  ? sample-size: uint
  ? generated: uint
  ? expires: uint
  * text => any
}
~~~~
{:cddl}

Note how this rather clearly delineates the structure somewhat
shrouded by so many words in section 6.2.2. of {{RFC7071}}.
Also, this definition makes it clear that several ext-values are
allowed (by definition with different member names); RFC 7071 could be
read to forbid the repetition of ext-value ("A specific
reputon-element MUST NOT appear more than once" is ambiguous.)

The CDDL tool (which hasn't quite been trained for polite conversation)
says:

~~~~ JSON
{
  "application": "tridentiferous",
  "reputons": [
    {
      "rater": "loamily",
      "assertion": "Dasyprocta",
      "rated": "uncommensurableness",
      "rating": 0.05055809746548934,
      "confidence": 0.7484706448605812,
      "normal-rating": 0.8677887734049299,
      "sample-size": 4059,
      "expires": 3969,
      "bearer": "nitty",
      "faucal": "postulnar",
      "naturalism": "sarcotic"
    },
    {
      "rater": "precreed",
      "assertion": "xanthosis",
      "rated": "balsamy",
      "rating": 0.36091333590593955,
      "confidence": 0.3700759808403371,
      "sample-size": 3904
    },
    {
      "rater": "urinosexual",
      "assertion": "malacostracous",
      "rated": "arenariae",
      "rating": 0.9210673488013762,
      "normal-rating": 0.4778762617112776,
      "sample-size": 4428,
      "generated": 3294,
      "backfurrow": "enterable",
      "fruitgrower": "flannelflower"
    },
    {
      "rater": "pedologistically",
      "assertion": "unmetaphysical",
      "rated": "elocutionist",
      "rating": 0.42073613384304287,
      "misimagine": "retinaculum",
      "snobbish": "contradict",
      "Bosporanic": "periostotomy",
      "dayworker": "intragyral"
    }
  ]
}
~~~~
{:cddl}

## Examples from JSON Content Rules

Although [JSON Content Rules](#I-D.newton-json-content-rules) seems to
address a more general problem than CDDL, it is still a worthwhile
resource to explore for examples (beyond all the inspiration the
format itself has had for CDDL).

Figure 2 of the JCR I-D looks very similar, if slightly less noisy, in CDDL:

~~~~ CDDL
root = [2*2 {
  precision: text,
  Latitude: float,
  Longitude: float,
  Address: text,
  City: text,
  State: text,
  Zip: text,
  Country: text
}]
~~~~
{:cddl}
{: #jcrfig2 title="JCR, Figure 2, in CDDL"}

Apart from the lack of a need to quote the member names, text strings
are called `text` or `tstr` in CDDL (`string` would be ambiguous as
CBOR also provides byte strings).

The CDDL tool creates the below example instance for this:

~~~~ CBORdiag
[{"precision": "pyrosphere", "Latitude": 0.5399712314350172,
  "Longitude": 0.5157523963028087, "Address": "resow",
  "City": "problemwise", "State": "martyrlike", "Zip": "preprove",
  "Country": "Pace"},
 {"precision": "unrigging", "Latitude": 0.10422704368372193,
  "Longitude": 0.6279808663725834, "Address": "picturedom",
  "City": "decipherability", "State": "autometry", "Zip": "pout",
  "Country": "wimple"}]
~~~~
{:cddl}

Figure 4 of the JCR I-D in CDDL:

~~~~ CDDL
root = { image }

image = (
  Image: {
    size,
    Title: text,
    thumbnail,
    IDs: [* int]
  }
)

size = (
  Width: 0..1280
  Height: 0..1024
)

thumbnail = (
  Thumbnail: {
    size,
    Url: uri
  }
)
~~~~
{:cddl}

This shows how the group concept can be used to keep related elements
(here: width, height) together, and to emulate the JCR style of
specification.  (It also shows using a tag from the prelude, `uri` -- this
could be done differently.)  The more compact form of Figure 5 of the
JCR I-D could be emulated like this:

~~~~ CDDL
root = {
  Image: {
    size, Title: text,
    Thumbnail: { size, Url: uri },
    IDs: [* int]
  }
}

size = (
  Width: 0..1280,
  Height: 0..1024,
)
~~~~
{:cddl}


The CDDL tool creates the below example instance for this:

~~~~ CBORdiag
{"Image": {"Width": 566, "Height": 516, "Title": "leisterer",
  "Thumbnail": {"Width": 1111, "Height": 176, "Url": 32("scrog")},
  "IDs": []}}
~~~~
{:cddl}

# Making Use of CDDL {#usage}

In this section, we discuss several potential ways to employ CDDL.

## As a guide to a human user

CDDL can be used to efficiently define the layout of CBOR data,
such that a human implementer can easily see how data is supposed to be encoded.

Since CDDL maps parts of the CBOR data to human readable names,
tools could be built that use CDDL to provide a human friendly representation
of the CBOR data,
and allow them to edit such data while remaining compliant to its CDDL definition.

## For automated checking of CBOR data structure

CDDL has been specified such that a machine can handle the CDDL definition
and related CBOR data.
For example,
a machine could use CDDL to check whether or not CBOR data is compliant
to its definition.

The need for thoroughness of such compliance checking depends on the application.
For example,
an application may decide not to check the data structure at all,
and use the CDDL definition solely as a means to indicate the structure of
the data to the programmer.

On the other end,
the application may also implement a checking mechanism that goes as
far as checking that all mandatory map pairs are available.

The matter in how far the data description must be enforced by an application
is left to the designers and implementers of that application,
keeping in mind related security considerations.

In no case the intention is that a CDDL tool would be "writing code" for
an implementation.

## For data analysis tools

In the long run,
it can be expected that more and more data will be stored using the CBOR
data format.

Where there is data, there is data analysis and the need to process such
data automatically.
CDDL can be used for such automated data processing,
allowing tools to verify data, clean it,
and extract particular parts of interest from it.

Since CBOR is designed with constrained devices in mind,
a likely use of it would be small sensors.
An interesting use would thus be automated analysis of sensor data.

--- back

# Cemetery

The following ideas have been buried in the discussions leading up to
the present specification:

* <...> as syntax for enumerations. We view values to be just another
type (a very specific type with just one member), so that an
enumeration can be denoted as a choice using "/" as the delimiter of
choices. Because of this, no evidence is present that a separate
syntax for enumerations is needed.

## Resolved Issues

* The key/value pairs in maps have no fixed ordering.  One could
imagine situations where fixing the ordering may be of use.  For
example, a decoder could look for values related with integer keys 1,
3 and 7.  If the order were fixed and the decoder encounters the key 4
without having encountered key 3, it could conclude that key 3 is not
available without doing more complicated bookkeeping.
Unfortunately, neither JSON nor CBOR support this, so no attempt was
made to support this in CDDL either.

* CDDL distinguishes the various CBOR number types, but there is
only one number type in JSON.  There is no effect in specifying a
precision (float16/float32/float64) when using CDDL for specifying
JSON data structures.  (The current validator implementation {{tool}} does not
handle this very well, either.)

# Nursery

This appendix describes advanced features that are still under review
as they have not yet been heavily used in specifications.

## Generics

Using angle brackets, the left hand side of a rule can add formal
parameters after the name being defined, as in:

~~~ CDDL
messages = message<"reboot", "now"> / message<"sleep", 1..100>
message<t, v> = {type: t, value: v}
~~~
{:cddl}

When using a generic rule, the formal parameters are bound to the
actual arguments supplied (also using angle brackets), within the
scope of the generic rule (as if there were a rule of the form
parameter = argument).

(There are some limitations to nesting of generics in {{tool}} at this
time.)

# Change Log

Changes from version 00 to version 01:

* Removed constants

* Updated the tag mechanism

* Extended the map structure

* Added examples


Changes from version 01 to version 02:

* Fixed example


Changes from version 02 to version 03:

* Added information about characters used in names

* Added text about an overall data structure and order of definition of fields

* Added text about encoding of keys

* Added table with keywords

* Strings and integer writing conventions

* Added ABNF


Changes from version 03 to version 04:

* Removed optional fields for non-maps

* Defined all key/value pairs in maps are considered optional from the CDDL
perspective

* Allow omission of type of keys for maps with only text string and integer
keys

* Changed order of definitions

* Updated fruit and moves examples

* Renamed the "Philosophy" section to "Using CDDL", and added more text about
CDDL usage

* Several editorials


Changes from version 04 to version 05:

* Added text about alternative datatypes and any datatype

* Fixed typos

* Restructured syntax and semantics


Changes from version 05 to version 05:

* Fixed the ABNF for choices (no longer need to write a: (b/c))

* Added group choices (//)

* Added /= and //=

* Added experimental socket/plug

* Added aliases text, bytes, null to prelude

* Documented generics

* Fixed more typos

Changes from 06 to 07:

* .cbor, .cborseq, .within, .and

* Define .size on uint

* Extended Diagnostic Notation

* Precedence discussion and table

* Remove some of the "issues" that can only be understood with
  historical context

* Prefer `text` over `tstr` in some of the examples

* Add `unsigned` to the prelude

Changes from 07 to 08:

* .lt, .le, .eq, .ne, .gt, .ge
* .default

Changes from 08 to 09:

* Take annotations and socket/plug out of the nursery; they have been
  battle-proven enough.
* Define a value notation for byte strings as well.
* Removed discussion section that was no longer relevant; move
  "Resolved Issues" to appendix.

Changes from 09 to 10:

* Remove a long but not very elucidating example.  (Maybe we'll add
  back some shorter examples later.)
* A few clarifications.
* Updated author list.

--- middle

# Security considerations

This document presents a content rules language for expressing CBOR data
structures.
As such,
it does not bring any security issues on itself,
although specification of protocols that use CBOR naturally need security
analysis when defined.

Topics that could be considered in a security considerations section that
uses CDDL to define CBOR structures include the following:

* Where could the language maybe cause confusion in a way that will
  enable security issues?

# IANA considerations

This document does not require any IANA registrations.

# Acknowledgements

CDDL was originally conceived by Bert Greevenbosch, who also wrote the
original five versions of this document.

Inspiration was taken from the C and Pascal languages, MPEG's
conventions for describing structures in the ISO base media file
format, Relax-NG and its compact syntax {{RELAXNG}}, and in particular
from Andrew Lee Newton's ["JSON Content Rules"](#I-D.newton-json-content-rules).

Useful feedback came from Joe Hildebrand, Sean Leonard and
Jim Schaad.

The CDDL tool was written by Carsten Bormann, building on previous
work by Troy Heninger and Tom Lord.

--- back

# ABNF grammar {#abnf}

The following is a formal definition of the CDDL syntax in Augmented Backus-Naur Form
(ABNF, {{RFC5234}}).[^_abnftodo]


~~~~ ABNF
{::include cddl.abnf}
~~~~
{:cddl #fig-abnf title="CDDL ABNF"}

[^_abnftodo]: TO DO: This doesn't allow non-ASCII characters in the
        text or byte strings yet (and the prefixed byte strings are
        more liberally specified than they actually are);
        representation indicators are missing as well.

# Standard Prelude {#prelude}

The following prelude is automatically added to each CDDL file[^tdate].
(Note that technically, it is a postlude, as it does not disturb the
selection of the first rule as the root of the definition.)

~~~~ CDDL
{::include cddl.prelude}
~~~~
{:cddl #fig-prelude title="CDDL Prelude"}

Note that the prelude is deemed to be fixed.  This means, for instance,
that additional tags beyond {{RFC7049}}, as registered, need to be
defined in each CDDL file that is using them.

A common stumbling point is that the prelude does not define a type
`string`.  CBOR has byte strings (`bytes` in the prelude) and text
strings (`text`), so a type that is simply called `string` would be
ambiguous.

[^tdate]: The prelude as included here does not yet have a .regexp
        annotation on tdate, but we probably do want to have one.

# The CDDL tool {#tool}

A rough CDDL tool is available.  For CDDL specifications that do not
use recursion, it can check the syntax, generate one or more instances
(expressed in CBOR diagnostic notation or in pretty-printed JSON), and
validate an existing instance against the specification:

~~~~
Usage:
cddl spec.cddl generate [n]
cddl spec.cddl json-generate [n]
cddl spec.cddl validate instance.cbor
cddl spec.cddl validate instance.json
~~~~
{:cddl #tool-usage title="CDDL tool usage"}

Install on a system with a modern Ruby via:

~~~~ shell
gem install cddl
~~~~
{:cddl #tool-install title"CDDL tool installation"}


The accompanying CBOR diagnostic tools (which are automatically
installed by the above) are described in
<https://github.com/cabo/cbor-diag>; they can be used to convert between
binary CBOR, a pretty-printed form of that, CBOR diagnostic notation,
JSON, and YAML.

# Extended Diagnostic Notation

Section 6 of {{RFC7049}} defines a "diagnostic notation" in order to
be able to converse about CBOR data items without having to resort to
binary data.  Diagnostic notation is based on JSON, with extensions
for representing CBOR constructs such as binary data and tags.

(Standardizing this together with the actual interchange format does
not serve to create another interchange format, but enables the use of
a shared diagnostic notation in tools for and documents about CBOR.)

This section discusses a few extensions to the diagnostic notation
that have turned out to be useful since RFC 7049 was written.
We refer to the result as extended diagnostic notation (EDN).

## White space in binary strings

Examples often benefit from some white space (spaces, line breaks) in
binary strings.  In extended diagnostic notation, white space is
ignored in prefixed binary strings; for instance, the following are
equivalent:

      h'48656c6c6f20776f726c64'
      h'48 65 6c 6c 6f 20 77 6f 72 6c 64'
      h'4 86 56c 6c6f
        20776 f726c64'

## Text in binary strings {#textbin}

Diagnostic notation notates Byte strings in one of the {{?RFC4648}}
base encodings,, enclosed in single quotes, prefixed by >h< for
base16, >b32< for base32, >h32< for base32hex, >b64< for base64 or
base64url.  Quite often, binary strings carry bytes that are
meaningfully interpreted as UTF-8 text.  Extended Diagnostic Notation
allows the use of single quotes without a prefix to express byte
strings with UTF-8 text; for instance, the following are equivalent:

    'hello world'
    h'68656c6c6f20776f726c64'

The escaping rules of JSON strings are applied equivalently for
text-based binary strings, e.g., \\ stands for a single backslash and
\' stands for a single quote.  White space is included literally,
i.e., the previous section does not apply to text-based binary strings.

## Concatenated Strings

While the ability to include white space enables line-breaking of
encoded binary strings, a mechanism is needed to be able to include
text strings as well as binary strings in direct UTF-8 representation
into line-based documents (such as RFCs and source code).

We extend the diagnostic notation by allowing multiple text strings or
multiple byte strings to be notated separated by white space, these
are then concatenated into a single text or byte string, respectively.
Text strings and binary strings do not mix within such a
concatenation, except that binary string notation can be used inside a
sequence of concatenated text string notation to encode characters
that may be better represented in an encoded way.  The following four
values are equivalent:

    "Hello world"
    "Hello " "world"
    "Hello" h'20' "world"
    "" h'48656c6c6f20776f726c64' ""

Similarly, the following byte string values are equivalent

    'Hello world'
    'Hello ' 'world'
    'Hello ' h'776f726c64'
    'Hello' h'20' 'world'
    '' h'48656c6c6f20776f726c64' '' b64''
    h'4 86 56c 6c6f' h' 20776 f726c64'

(Note that the approach of separating by whitespace, while familiar
from the C language, requires some attention -- a single comma makes a
big difference here.)

## Hexadecimal, octal, and binary numbers

In addition to JSON's decimal numbers, EDN provides hexadecimal, octal
and binary numbers in the usual C-language notation (octal with 0o
prefix present only).

The following are equivalent:

    4711
    0x1267
    0o11147
    0b1001001100111

As are:

    1.5
    0x1.8p0
    0x18p-4

## Comments

Longer pieces of diagnostic notation may benefit from comments.
JSON famously does not provide for comments, and basic RFC 7049
diagnostic notation inherits this property.

In extended diagnostic notation, comments can be included, delimited
by slashes ("/").  Any text within and including a pair of slashes is
considered a comment.

Comments are considered white space.  Hence, they are allowed in
prefixed binary strings; for instance, the following are equivalent:

    h'68656c6c6f20776f726c64'
    h'68 65 6c /doubled l!/ 6c 6f /hello/
      20 /space/
      77 6f 72 6c 64' /world/

This can be used to annotate a CBOR structure as in:

~~~~ CBORdiag
/grasp-message/ [/M_DISCOVERY/ 1, /session-id/ 10584416,
                 /objective/ [/objective-name/ "opsonize",
                              /D, N, S/ 7, /loop-count/ 105]]
~~~~

(There are currently no end-of-line comments.  If we want to add them,
"//" sounds like a reasonable delimiter given that we already use
slashes for comments, but we also could go e.g. for "#".)

<!--  LocalWords:  notational CBOR CDDL JSON valuetype datatype ABNF IP
 -->
<!--  LocalWords:  valuetypes datatypes processable tstr uint nint LF
 -->
<!--  LocalWords:  bstr keytype Struct ASN positionally struct Structs
 -->
<!--  LocalWords:  optionality extensibility structs predefines CRLF
 -->
<!--  LocalWords:  bareword barewords UTF bool IEEE reputon analysis
 -->
<!--  LocalWords:  wildcard validator IANA
 -->
