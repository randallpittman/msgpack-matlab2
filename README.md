# MessagePack for MATLAB - MEX bindings for msgpack-c supporting the modern MessagePack standard

Forked from [msgpack-matlab by yida](https://github.com/yida/msgpack-matlab).

## Purpose
To provide a MEX interface to the msgpack-c library (https://github.com/msgpack/msgpack-c) that supports [the modern MessagePack specification](https://github.com/msgpack/msgpack/blob/master/spec.md).

**yida/msgpack-matlab** provided an excellent foundation for this project, but was lacking in the following areas:
- Only supported legacy MessagePack (see msgpack/msgpack/spec-old.md) (no BIN or EXT types, non-unicode RAW/STR)
- Raw/String support did not handle non-ascii bytes well.
- 

MEX bindings for msgpack-c (https://github.com/msgpack/msgpack-c)

Requires:
* msgpack-c development library > 1.1.0 (for unpacking FLOAT32 to single need msgpack-c > 2.1.0)

install: 

```bash
mex -O msgpack.cc -lmsgpack
```

## API

### Packer:

```matlab
>> msg = msgpack('pack', var1, var2, ...)
```

### Unpacker:

```matlab
>> obj = msgpack('unpack', msg) 
```

return numericArray or charArray or LogicalArray if data are numeric otherwise return Cell or
 Struct
  
### Streaming unpacker:

```matlab
>> objs = msgpack('unpacker', msg)
```

return Cell containing numericArray, charArray, Cell or Struct

### Packing EXT type
A 1x3 cell array `{'MSGPACK_EXT', <ext_code>, <data_bytes_uint8>}` will be packed as EXT type.

### Flags

Flags may be set that affect this and future calls of `msgpack()` as follows:

```matlab
>> msgpack('<cmd> [<flag>[ <flag>[ ...]]]', ...)
```

...where `<cmd>` may be simply `set_flags` or one of the other commands followed by that command's
 arguments, and where each `<flag>` is one of the following, prefixed with `+` to set or `-` to
 unset:

* `+unicode_strs` or `-unicode_strs` (default is **set**)
  * **Set** - MessagePack strings are assumed to be UTF-8 and are unpacked to MATLAB's UTF-16, 
    and vis-versa
  * **Unset** - MessagePack strings are of unknown encoding and are unpacked as a uint8 array.
    When packing a `char` array, just try to pack MATLAB `mxChar`s if they are smaller than
    `0x00ff` into the MessagePack string field.
* `+pack_u8_bin` or `-pack_u8_bin` (default is **unset**)
  * **Set** - MATLAB `uint8` arrays are packed into MessagePack bin messages.
  * **Unset** - MATLAB `uint8` arrays are packed into MessagePack arrays.
  * In both cases, non-`uint8` arrays are packed into MessagePack arrays.
    Use MATLAB's `typecast` to convert to `uint8` to pack other types as
    MessagePack bin messages.
* `+unpack_narrow` or `-unpack_narrow` (default is **unset**)
  * **Set** - When unpacking scalar numeric types, unpack to the narrowest type possible.
    * Positive integers -> smallest possible uint
    * Negative integers -> smallest possible int
    * Float32 -> single
    * Float64 -> double
  * **Unset** - Unpack all scalar numeric types to MATLAB double type.
  * In both cases, arrays of all the same numeric type are converted as follows:
    * Positive integers -> uint64
    * Negative integers -> int64
    * Float32 -> single
    * Float64 -> double
* `+unpack_map_as_cells` or `-unpack_map_as_cells` (default is **unset**)
  * **Set** - When unpacking a map, always unpack as a 2xN cell matrix of keys and values.
  * **Unset** - If all keys are strings, unpack a map to a struct. If not, generate a
    warning and unpack to a cell matrix as above.
* `+unpack_ext_w_tag` or `-unpack_ext_w_tag` (default is **unset**)
  * **Set** - When unpacking an ext type, unpack to 1x3 cell matrix of
    `{'MSGPACK_EXT', <ext_code>, <data_byytes_uint8>}`
  * **Unset** - When unpacking an ext type, unpack to 1x2 cell matrix of `{<ext_code>, <data>}`
* `+pack_other_as_nil` or `-pack_other_as_nil` (default is **unset**)
  * **Set** - If trying to pack a type for which there is no packer function, show a warning and
    pack NIL instead.
  * **Unset** - If trying to pack a type for which there is no packer function, throw an error.
* `+unpack_nil_zero`, `+unpack_nil_NaN`, `+unpack_nil_empty`, `+unpack_nil_cell`
  (Default is `unpack_nil_zero`)
  * Rather than **set** and **unset**, there are multiple options for how to unpack NIL type.
  * `zero` - Unpack to zero, (or false in logical arrays). Double if scalar, uint8 if all-nil array.
  * `NaN` - Unpack to NaN. In otherwise integer or logical array forces a cell array.
  * `empty` - Unpack to empty double array. In an array forces a cell array.
  * `cell` - Unpack to empty cell array. In an array forces the array to a cell array.
* `+unpack_nil_array_skip` or `-unpack_nil_skip` (default is **unset**)
  * **Set** - If a `nil` is in an otherwise numeric or logical array, skip the `nil`. If all-nils,
              return an empty array.
  * **Unset** - Unpack `nil` as in `unpack_nil_...` above.

To reset flags to defaults:
```matlab
msgpack('reset_flags');
```

To see the currently set flags
```matlab
msgpack('print_flags');
```
