# CDDL Review

## Chapter 2

2.2.1, conflicting definitions?
Compare
Values such as numbers and strings can be used in place of a type.

to
Each value that can be expressed as a CBOR data item is also a type in its own right, e.g., "1".

"in place of" is different to "a type in its own right", isn't it?

2.2.2
change "added to a rule later in separate rules" to "added to a rule later on in separate rules"

(OLD) be added to a rule later in separate rules
(NEW) be added to a rule later on in separate rules

## Chapter 3

3.2 Occurences example with apartment

(OLD) size = float ; in m2
(NEW) size = float ; in square metres (m2)

3.5.1 Structs
Simplify this sentence
(OLD) A map is defined in the same way as that for defining an array (see Section 3.4), except for using curly braces "{}" instead of square brackets "[]".
(NEW) A map is defined in the same way as an array is defined (see Section 3.4), except for using curly braces "{}" instead of square brackets "[]".

"structure" vs "struct", use struct consistently as we refer to the type of composition, multiple occurences:
(OLD)    The following is an example of a record with a structure embedded:
(NEW)    The following is an example of a record with a struct embedded:

(OLD)    whereas the GpsCoordinates structure is encoded as a CBOR map with
(NEW)    whereas the GpsCoordinates struct is encoded as a CBOR map with

Is the occurence explanation still neccessary? Shouldn't we explain the example more, something like
(OLD) In this example, all key/value pairs are optional from the perspective of CDDL.  With no occurrence indicator, an entry is mandatory.
(BETTER) In this example, all key/value pairs are optional from the perspective of CDDL.  With no occurrence indicator, the group NameComponents is included mandatorily, but its group items are optional.

3.5.3. Non-Deterministic Ordering
Correct a typo
(OLD) all remaining text/number member will be picked by the second entry (and if anything remains unpicked, the map does not match).
(NEW) all remaining text/number members will be picked by the second entry (and if anything remains unpicked, the map does not match).


3.8.4 Control Operators .cbor and .cborseq
type1 and type2 are used already in the ABNF and in other parts of the document, referring to their respective rules in the ABNF. To reduce potential for confusion, change type1 and type2 to typeL and typeR and use them for their actual position when used with the operators:
(OLD) A ".cbor" control on a byte string indicates that the byte string
   carries a CBOR-encoded data item.  Decoded, the data item matches the
   type given as the right-hand-side argument (type1 in the following
   example).

      "bytes .cbor type1"


   Similarly, a ".cborseq" control on a byte string indicates that the
   byte string carries a sequence of CBOR-encoded data items.  When the
   data items are taken as an array, the array matches the type given as
   the right-hand-side argument (type2 in the following example).

      "bytes .cborseq type2"
(NEW) A ".cbor" control on a byte string indicates that the byte string
   carries a CBOR-encoded data item.  Decoded, the data item matches the
   type given as the right-hand-side argument (typeR in the following
   example).

      "bytes .cbor typeR"


   Similarly, a ".cborseq" control on a byte string indicates that the
   byte string carries a sequence of CBOR-encoded data items.  When the
   data items are taken as an array, the array matches the type given as
   the right-hand-side argument (typeR in the following example).

      "bytes .cborseq typeR"

3.8.5 Control Operators .within and .and
type1 and type2 are used already in the ABNF and in other parts of the document, referring to their respective rules in the ABNF. To reduce potential for confusion, change type1 and type2 to typeL and typeR and use them for their actual position when used with the operators:
(OLD)       "type1 .and type2"
(NEW)       "typeL .and typeR"

(OLD)       "type1 .within type2"
(NEW)       "typeL .within typeR"

(OLD)    For ".within", a tool might flag an error if type1 allows data items that are not allowed by type2.  In contrast, for ".and", there is no expectation that type1 is already a subset of type2.
(NEW)    For ".within", a tool might flag an error if typeL allows data items that are not allowed by typeR.  In contrast, for ".and", there is no expectation that typeL is already a subset of typeR.

## Chapter 5
5. Security Considerations
Last point is unclear without an added "specification", isn't it?
(OLD)    o  Where the CDDL includes extension points, the impact of extensions on the security of the system needs to be carefully considered.
(NEW)    o  Where the CDDL specification includes extension points, the impact of extensions on the security of the system needs to be carefully considered.
