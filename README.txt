BSF: The Big Simple Format file specification

version 0.0.1 <add official release date here when satisfied>

This document defines the Basic Simple Format file specification.

Aims: define a simple to read/write format for numeric data. Accenting
the simple part of the definition the data is laid out in a regular/fixed
format that is n-dimensional and uncompressed. (A simple file format
should not require readers and writers to be complex enough to support
compression)

All numbers are stored in high byte first order (Big endian)

Example pseudo format:

"BSF" : file type marker string
<15-bit unsigned number> (Major version)
<15-bit unsigned number> (Minor version)
<15-bit unsigned number> (Subminor version)
<some set of (C?) string key value pairs of metadata>
number of dimensions (unsigned 63 bit type)
dimension granularity bit count (all dimensions are unsigned). This field
  describes the number of bits each following dimension number takes up.
  Valid values are 1 to 65536. Each number is encoded to the nearest
  8-bit boundary. So a value of 5 says store a 5 bit number in the next
  8 bits.
dimension 0 : (a number of size specified by the within the granularity
 specified above)
equation string 0 (and an equation string specifying calibration)
  it's a C? string. calibration can be ignored if desired. if calibration
  is specified the equation language is specified below.
...
dimension n-1 : (a number within the granularity specified above)
equation string n-1 (and an equation string specifying calibration)
<the type of numeric data entries that follow in subentities
  INT [numbits between 1 and 65536]
  UINT [numbits between 1 and 65536]
  IEEE_FLOAT [numbits == 8? or 16 or 32 or 64 or 128]
  (some day: POSIT posit descriptors of some kind)
>
<subentity type: 1 (REAL) or 2 (COMPLEX) or 4 (QUATERNION) or 8 (OCTONION)>
<entity dimension count: UINT15 0 = number, 1 = vector, 2 = matrix, 3+ = tensor>
<entity dimension granularity bit count: valid values are 1 to 2^64-1
<entity dimension 0 : within entity dimension granularity>
...
<entity dimension n-1 : within entity dimension granularity>
<all the numeric data follows here sequentially. the order of elements
 is dimension0/dimension1/dimension2/etc and within then by entity and
 then by subentity. I will soon make an example that shows the order.
 the dimension indices increase leftmost first. the entities also go
 subentity dim 0 to subentity dim n-1.>
 
   Example code that would work to read the data portion:
   
     // read the data header values
     
     for (uint63 i = 0; i < NUMDIMS; i++) {
       read and store DIM value i
       read and store DIM calibration equation string
     }
     read and store the subentity component type (such as INT12, or FLOAT64, etc)
     read and store the number of components that make up a sub entity (1, 2, 4, or 8)
     read and store the entity type dimension count
        (i.e. 2 says the entities are 2-d matrices of 1, 2, 4, or 8 sub entities each)
     read and store the ENTITY dim granularity (UINT63)
     for i = 0 < entity type dim count
       read and store the ENTITY_DIM
       
     // now read the data
     
     total entities = each DIM multiplied together
     for bignum i = 0; i < tot entities; i++
       // read an entity
       for uint63 d = 0; d < entity dim count
         for the dims of the entity (for instance a 3x4 matrix)
           for (se = 0; se < 1 or 2 or 4 or 8; se++) {
             read a subentity component value (like the 1st component of a cmplx num)
           }
           make a sub entity from those read values (like a quat<float32>)
           store the subentity at the right position within the entity
             (e.g set vector[7] = subentity read)

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
- should I define a fixed record format so that a record can be made of
    a set of mixed types? The format would still be a bunch of fixed
    records but not just of one kind of number. This would allow CMYK or
    YUV etc. but there is no inferring from a record type that it is a
    CMYK etc.
