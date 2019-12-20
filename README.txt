BSF: The Big Simple Format file specification

version 0.0.1

This document defines the Basic Simple Format file specification.

Aims: define a simple to read format for numeric data. Accenting the
simple part of the definition the data is laid out in a regular/fixed
format that is n-dimensional and uncompressed. (A simple file format
should not require readers and writers to be complex enough to support
compression)

Example pseudo format:

BSF
<version (C?) string i.e. "0.0.1" right now>
<some set of (C?) string key value pairs of metadata>
number of dimensions (UINT63?)
dimension granularity (UINT12, UINT31, UINT63, etc. up to UINT65536)
dimension 0 : (a number within the granularity specified above) (and an
 equation string specifying calibration)
...
dimension n-1 : (a number within the granularity specified above) (and
 an equation string specifying calibration)
<the type of numeric data entries that follow in subentities
  INT [numbits]
  UINT [numbits]
  IEEE_FLOAT [numbits == 8? or 16 or 32 or 64 or 128]
  (some day: POSIT posit descriptors)
>
<subentity type: REAL(1) or COMPLEX(2) or QUATERNION(4) or OCTONION(8)>
<entity dimension count: 0 = number, 1 = vector, 2 = matrix, 3+ = tensor>
<entity dimension 0 : UINT63>
...
<entity dimension n-1 : UINT63>
<all the numeric data follows here sequentially. the order of elements
 is dimension0/dimension1/dimension2/etc and within then by entity and
 then by subentity. I will soon make an example that shows the order.
 the dimension indices increase leftmost first. the entities also go
 subentity dim 0 to subentity dim n-1.>

Notes/questions
- even though someone could be specifying UINT12's they take up two
    bytes. Again no form of compression is supported.
- all numeric values are to be specified as running from high byte to
    lo byte (big endian). This simplifies the reading of the numbers
    into arbitrary precision integers. You can read one byte and then
    shift left 8 bits and read the next byte etc. It also makes the
    format more simple to understand.
- equation language must be laid out so that one knows how to parse.
    for example specify what functions are valid (sin? acos? erfc? etc.)
    but the calibration strings can be ignored by anyone who wants to.
- ideally the numbers would be expandable: i.e. no fixed sizes (such as
    UINT63) for dim counts and dim sizes.
- supporting this format requires arbitrary precision ints for sizes and
    arbitrary precision floats for axis calibration / equation support.
- how complex should the subentity codes be. The basic couple I've
    defined? Or a ton (various color model formats, gaussian integers,
    others?). Ideally the entities can be identified enough that a
    program using this can know what mathematical operations are valid
    for the data. If you supported RGB and ARGB and YUV and CMYK and
    HSV/B you could reason what the 3 or 4 channels mean. Otherwise
    you'd just have one dimension that had value of 3 or 4 and you had
    no idea the data is RGB or ARGB or YUV etc. As I think more I believe
    that we should keep the type count minimized so no color model types.
- should I define a fixed record format so that a record can be of mixed
    types? The format would still be a bunch of fixed records but not
    just of one kind of number. This would allow CMYK or YUV etc. but
    there is no inferring from a record type that it is a CMYK etc.
