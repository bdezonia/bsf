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
<some affine definitin of the coordinate system?>
number of dimensions (unsigned 63 bit type)
dimension granularity byte count (all dimensions are unsigned). This
  field describes the number of 8-bit bytes each following dimension
  number is specified with. Valid values are 1 to 65536.
dimension 0 : (a number within the granularity specified above)
...
dimension n-1 : (a number within the granularity specified above)
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
    
    
MUCH LATER IDEAS

Let's simplify a lot

A BSF file is an n-dimensional set of data records.

Each BSF file contains one kind of data record type. The
BSF header defines the structure of the data records.

A record is described as ...
  a list of int32 codes
    binary "1": signed integer. followed by a number of bits.
    binary "2": unsigned integer. followed by a number of bits.
    binary "3": ieee 8 bit float
    binary "4": ieee 16 bit float
    binary "5": ieee 32 bit float
    binary "6": ieee 64 bit float
    binary "7": ieee 128 bit float
    binary "8": utf16 char array. followed by a number specifying the size of the array.
      will these chars pack well given that each char can be a different size?
    binary "-1" : end of record
    
a record definition is a list of consecutive codes 

And a default value for a record can be defined in the header.

  encoded matching the definition in the header
    (for instance someone could set to zeroes or NaN or blank strings etc)

  (it could be that when you define the record you also define the
    default value for each field instead of defnining a separate "zero"
    value record.)

a record definition also has a 512 char array name
        
A BSF file specifies it's number and size of dimensions.

All numbers in a BSF file are saved in big endian format.
All characters in a BSF file are saved as UTF16.

Each record in a BSF file is preceded with the coordinates
of the point. The coordinate system is that the origin of
the data is always at the zero point of every axis with
each axis growing away from 0 in a positive direction.

Since the file specifies a default values record you can
write sparse files by only enumerating points that have
non-default values. Everything else in the file will be
set to the default record value.

There are some reserved record names. You can define your
own record name or better yet register them here within
this repo. I will document them as possible.

Reserved record names:

are strings going to factor into this?
strX : X is any number and is the number of UTF16 chars in the string
intX  : X is any number and is the bits contained in the int
uintX  : X is any number and is the bits contained in the unsigned int
ieee16 : ieee float 16 (half precision)
ieee32 : ieee float 32 (single precision)
ieee64 : ieee float 64 (double precision)
ieee128 : ieee float 128 (quad precision)
rgb24 : a record of three 8 bit numbers representing an RGB color
rgb48 : a record of three 16 bit numbers representing an RGB color
rgb72 : a record of three 24 bit numbers representing an RGB color
rgb96 : a record of three 32 bit numbers representing an RGB color
argb32 : a record of four 8 bit numbers representing an ARGB color
argb64 : a record of four 16 bit numbers representing an ARGB color
argb96 : a record of four 24 bit numbers representing an ARGB color
argb128 : a record of four 32 bit numbers representing an ARGB color
yuv
cmyk
labcie
hsv or hsb or hsl or all of them
bcd?
decimal?
etc.

Header

BSFF
<int32>  : version of file format
<record definition>
<default record value>
<data records section>


The data section records is just

<dim 1 value><dim 2 value><...><dim n value><record value>

dims go from 0 to max-1. I think 0-based
is best for simple file reading into arrays.

And continues like this until the end of the file.

(One weakness of format: Due to sparseness a file that
just ends due to bad communications with disk or across
network maybe look complete. I think I need an EOF marker.>

eof marker could be -100. -100 is never a valid dimension


so how could you use this system to specify a record that
was a 4x4 matrix or a 10 element vector or some tensor?

  a "conglomerate" type that has a backing number type
  and n-dims. a vector would be 1 dim of numbers. a matrix
  would be two dims of numbers. a thensor would be n dims
  of data. note that the backing type could be strings too.
