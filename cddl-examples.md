
# Examples {#examples}

This appendix is for information only.

This section contains a few examples of structures defined using
CDDL.

The theme for the first example is taken from {{RFC7071}}, which
defines certain JSON structures in English.  For a similar example, it
may also be of interest to examine Appendix A of {{RFC8007}}, which
contains a CDDL definition for a JSON structure defined in the
main body of the RFC.

The second subsection in this appendix translates examples from
{{-jcr}} into CDDL.

These examples all happen to describe data that is interchanged in
JSON.  Examples for CDDL definitions of data that is interchanged in
CBOR can be found in {{-cose}}, {{-grasp}}, or {{-senml}}.

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

The CDDL tool reported on in {{tool}} generates as one example:

~~~~ JSON
{
  "application": "conchometry",
  "reputons": [
    {
      "rater": "Ephthianura",
      "assertion": "codding",
      "rated": "sphaerolitic",
      "rating": 0.34133473256800795,
      "confidence": 0.9481983064298332,
      "expires": 1568,
      "unplaster": "grassy"
    },
    {
      "rater": "nonchargeable",
      "assertion": "raglan",
      "rated": "alienage",
      "rating": 0.5724646875815566,
      "sample-size": 3514,
      "Aldebaran": "unchurched",
      "puruloid": "impersonable",
      "uninfracted": "pericarpoidal",
      "schorl": "Caro"
    },
    {
      "rater": "precollectable",
      "assertion": "Merat",
      "rated": "thermonatrite",
      "rating": 0.19164006323936977,
      "confidence": 0.6065252103391268,
      "normal-rating": 0.5187773690879303,
      "generated": 899,
      "speedy": "solidungular",
      "noviceship": "medicine",
      "checkrow": "epidictic"
    }
  ]
}
~~~~
{:cddl}

## Examples from JSON Content Rules ##

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

The CDDL tool reported on in {{tool}} creates the below example instance for this:

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
    Url: ~uri
  }
)
~~~~
{:cddl}

This shows how the group concept can be used to keep related elements
(here: width, height) together, and to emulate the JCR style of
specification.  (It also shows referencing a type by unwrapping a tag
from the prelude, `uri` -- this could be done differently.)  The more
compact form of Figure 5 of the JCR I-D could be emulated like this:

~~~~ CDDL
root = {
  Image: {
    size, Title: text,
    Thumbnail: { size, Url: ~uri },
    IDs: [* int]
  }
}

size = (
  Width: 0..1280,
  Height: 0..1024,
)
~~~~
{:cddl}


The CDDL tool reported on in {{tool}} creates the below example instance for this:

~~~~ CBORdiag
{"Image": {"Width": 566, "Height": 516, "Title": "leisterer",
  "Thumbnail": {"Width": 1111, "Height": 176, "Url": 32("scrog")},
  "IDs": []}}
~~~~
{:cddl}

