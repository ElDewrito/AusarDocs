# Cached Tag Format

Cached tag data is at the heart of the Blam engine's content system. Any engine subsystem can define data which it is able to access at runtime through a group tag.
During build time, tag data is compiled into raw memory structures which are then dumped ("cached") to a file.
This makes loading very efficient because the data simply needs to be copied into a memory buffer and then fixed up (by finalizing memory pointers, etc.).
In Ausar, cached tag data is stored in files inside modules.

For an overview of how this system works in Halo 2, see this fantastic GDC 2005 presentation by Bungie: [Content Management for Halo 2 and Beyond](http://nikon.bungie.org/misc/gdc2005_mnoguchi/) ([archive link](https://archive.fo/Q9TyH)).

## Major Differences from Third-Generation Engines

*(We define third-generation engines to be the Blam engines used in Xbox 360 games.)*

There are several major differences between cached tag data in Ausar and the cached tag data in third-generation engines (.map files).
It is important to understand these differences in order to work with tag data effectively.

* All data is little-endian.
* All pointers are now 64 bits wide because the engine is 64-bit.
* Pointers are not stored in tag data in any form. They are written in at runtime based on data block locations.
* Tag data is now broken into separate blocks. All addressing is done through a block index and an offset into the block.
* Tags are identified at load time through a "global ID" and at runtime by a separate auto-generated "local handle". The process for generating the local handle has not been determined yet.
* Every tag has its own string table which is used to store referenced tag names and `string_id` values.
* `string_id` values are now Murmur3_32 hashes of their strings (instead of namespace+index pairs).
* The structures of many tag fields (tag blocks, data references, tag references, paged resource references) have changed significantly. Do not make _any_ assumptions about their structures without reading this document first. For example, tag references are no longer 16 bytes large and the group tag is no longer at the beginning. The new structures are documented below.

## General File Layout

Cached tag files have many different sections in them. Each section immediately follows the previous one. The location of a section is simply the sum of the sizes of all the sections before it.

The sections, in order, are:

* File header
* Tag dependency list
* Data block list
* Tag struct list
* Data reference list
* Tag reference list
* `string_id` list
* String table
* Zoneset information
* Tag data
* Resource data

(Note: when tags are stored in module files, everything through zoneset information is in one compressed block, the tag data is in another compressed block, and resource data is in multiple compressed blocks.)

## File Header

The file header is primarily responsible for holding the lengths of all of the sections which follow it.

### File Header Structure (Size = 0x50)

Offset | Type | Name | Description
--- | --- | --- | ---
0x00 | int32 | `magic` | `'hscu'` (UCSH = "UnCompreSsed Header"?)
0x04 | int32 | `version` | 27 (0x1B)
0x08 | uint8[0x14] | `unknown8` | *(Possibly a SHA-1 hash, really need to look into this)*
0x1C | int32 | `dependencyCount` | Number of tag dependencies
0x20 | int32 | `dataBlockCount` | Number of data blocks
0x24 | int32 | `tagStructCount` | Number of tag structs
0x28 | int32 | `dataReferenceCount` | Number of data references
0x2C | int32 | `tagReferenceCount` | Number of tag references
0x30 | int32 | `stringIdCount` | Number of `string_id`s
0x34 | uint32 | `stringTableSize` | String table size in bytes
0x38 | uint32 | `zonesetDataSize` | Zoneset data size in bytes
0x3C | uint32 | `headerSize` | Header size in bytes
0x40 | uint32 | `dataSize` | Tag data size in bytes
0x44 | uint32 | `resourceDataSize` | Resource data size in bytes
0x48 | int8 | `unknown48` |
0x49 | int8 | `unknown49` |
0x4A | int8 | `unknown4A` |
0x4B | int8 | `unknown4B` |
0x4C | int32 | `unknown4C` |

## Tag Dependency List

The tag dependency list is a list of all of the other tags that the current tag depends on. Tags are mainly identified through their global ID, but the dependency list stores other identification information too.

### Tag Dependency Structure (Size = 0x18)

Offset | Type | Name | Description
--- | --- | --- | ---
0x00 | int32 | `groupTag` | Group tag (e.g. `'proj'`).
0x04 | uint32 | `nameOffset` | Offset of the tag filename in the String Table.
0x08 | int64 | `assetId` | Asset ID of the tag.
0x10 | int32 | `globalId` | Global ID of the tag.
0x14 | int32 | `unknown14` |

## Data Block List

The data block list is a list of all the tag data blocks. This is used to locate each block within the tag data section of the file.

### Data Block Definition Structure (Size = 0x10)

Offset | Type | Name | Description
--- | --- | --- | ---
0x00 | uint32 | `size` | The size of the data block in bytes.
0x04 | int16 | `unknown4` |
0x06 | int16 | `unknown6` |
0x08 | uint64 | `offset` | The offset of the start of the data block, relative to the start of the tag data section of the file.

## Tag Struct List

The tag struct list is a list of all of the structs that appear in tag data, *including null references to structs*. It contains type information for each structure as well as instructions on how to fix up pointers to them. At load time, this is used to find the main structure and to write pointers into tag block and resource fields.

There *must* be at least one entry in this list (for the main struct) or else the engine will reject the tag file.

### Tag Struct Definition Structure (Size = 0x20)

Offset | Type | Name | Description
--- | --- | --- | ---
0x00 | uint8[0x10] | `guid` | Type GUID. This is used to write in a static pointer to type info located in the .rdata section of the executable. *(At load time, the first 8 bytes of this are replaced with the type info pointer for efficiency reasons.)*
0x10 | enum16 | `type` | 0 = Main Struct, 1 = Tag Block, 2 = Resource, 3 = Custom
0x12 | int16 | `unknown12` | *(May be padding, but haven't confirmed this)*
0x14 | int32 | `targetIndex` | For Main Struct and Tag Block structs, the index of the block containing the struct. For Resource structs, this (probably) is the index of the resource. This can be -1 if the tag field doesn't point to anything (null Tag Blocks or Custom structs).
0x18 | int32 | `fieldBlock` | The index of the data block containing the tag field which refers to this struct. Can be -1 for the Main Struct.
0x1C | uint32 | `fieldOffset` | The offset of the tag field inside the data block. (Unlike in Halo Online, this points to the tag field, *not* the pointer inside the tag field. Some tag fields for structs don't even have a pointer.)

### Main Struct (Type 0)

The main struct is the struct containing the root tag data. The Main Struct entry is usually at the beginning of the list and is probably used to locate the main tag data (need to confirm this though).

### Tag Blocks (Type 1)

Tag block entries refer to a tag block field that needs to be fixed up. At load time, the `targetIndex` is used to determine the final address of the data block containing the struct, and this 64-bit pointer is written to `fieldOffset` inside `fieldBlock`. The type info pointer is written to `fieldOffset + 0x8`.

### Resource Structs (Type 2)

Resource structs (presumably) refer to resource definition data. At load time, the `targetIndex` is (probably) used to determine the resource handle, and this handle is written to `fieldOffset + 0x8` inside `fieldBlock`. The type info pointer is written to `fieldOffset`.

### Custom Structs (Type 3)

Custom structs represent custom structures defined for the editor. No load-time fixups are done.

## Data Reference List

The data reference list is a list of all the data reference fields inside the tag data, *including null references*. It is used to fix up their pointers. Since data reference fields can refer to anything, they do not have formal tag data structures and therefore are not part of the tag struct list.

### Data Reference Definition Structure (Size = 0x14)

Offset | Type | Name | Description
--- | --- | --- | ---
0x00 | int32 | `parentStructIndex` | The index of the tag struct containing the tag field. (See the Tag Struct List file section.)
0x04 | int32 | `unknown4` |
0x08 | int32 | `targetIndex` | The index of the data block containing the referenced data. Can be -1 for null references.
0x0C | int32 | `fieldBlock` | The index of the data block containing the tag field.
0x10 | uint32 | `fieldOffset` | The offset of the tag field inside the data block.

### Fixups

At load time, the `targetIndex` is used to determine the final address of the data block containing the data, and this 64-bit pointer is written to `fieldOffset` inside `fieldBlock`. *(Some sort of type info pointer is written to `fieldOffset + 0x8`. Need to look into this.)*

## Tag Reference List

The tag reference list is a list of all tag reference fields inside the tag data, *including null references*. It is used to fix up their handles.

### Tag Reference Definition Structure (Size = 0x10)

Offset | Type | Name | Description
--- | --- | --- | ---
0x00 | int32 | `fieldBlock` | The index of the data block containing the tag field.
0x04 | uint32 | `fieldOffset` | The offset of the tag field inside the data block. (Unlike in Halo Online, this points to the tag field, *not* the handle inside the tag field.)
0x08 | uint32 | `nameOffset` | The offset of the tag filename within the String Table.
0x0C | int32 | `dependencyIndex` | The index of the tag dependency in the Tag Dependency List. Can be -1 for null tag references.

### Fixups

At load time, the `dependencyIndex` and global tag ID are used to determine the 32-bit local tag handle, which is written to `fieldOffset + 0x1C` inside `fieldBlock`. *(Some sort of type info pointer is written to `fieldOffset`. Need to look into this.)*

## `string_id` List

The `string_id` list is a list of all `string_id` values used by the tag. This does not store the actual values. Use Murmur3_32 on the actual strings to get them.

It is currently unknown if this section is actually used for anything at runtime.

### `string_id` Definition Structure (Size = 0x8)

Offset | Type | Name | Description
--- | --- | --- | ---
0x00 | uint32 | `unknown0` | *(Flags?)*
0x04 | uint32 | `stringOffset` | The offset of the string value within the String Table.

## String Table

The string table is a conglomeration of null-terminated ASCII strings for tag names and `string_id` strings. Other structures hold offsets into this.

## Zoneset Information

*(This needs to be looked into more to work out what exactly it's used for.)*

The Zoneset Information is a block of data which contains a list of zonesets that the tag belongs to. For each of these zonesets, there is a list of tag dependencies and the name of the zoneset to load for each of them. Many tags just have a single `__default__` zoneset. Look at a scenario tag for a good example of how this section is laid out.

Also, note that the header stores the size of this section in *bytes*, not the number of zonesets in it. This section has its own header which stores that value.

### Zoneset Information Header Structure (Size = 0x10)

This header is at the beginning of the Zoneset Information section.

Offset | Type | Name | Description
--- | --- | --- | ---
0x00 | int64 | `zonesetCount` | Number of zonesets.
0x08 | int64 | `zonesetListOffset` | Offset of the zoneset list from the start of the Zoneset Information section.

### Zoneset Entry Structure (Size = 0x28)

Arrays of these structures are pointed to by the `zonesetListOffset` in the Zoneset Information Header.

Offset | Type | Name | Description
--- | --- | --- | ---
0x00 | string_id | `name` | Name `string_id`.
0x04 | int32 | `unknown4` | *(Padding?)*
0x08 | int64 | `unknown8` |
0x10 | int64 | `unknown10` |
0x18 | int64 | `tagCount` | Number of tags to load for the zoneset.
0x20 | uint64 | `tagListOffset` | Offset of the tag list from the start of the Zoneset Information section.

### Zoneset Tag Structure (Size = 0x8)

Arrays of these structures are pointed to by the `tagListOffset` in each Zoneset Entry structure.

Offset | Type | Name | Description
--- | --- | --- | ---
0x00 | uint32 | `globalId` | Global ID of the tag.
0x04 | string_id | `zonesetName` | Name of the zoneset to load for the tag.

## Tag Data

Hey, we're finally here! *(And it only took Shockfire 2.5 straight hours of typing to get to this section!)*

This section contains the actual tag data in the file. It's stored in a separate compressed block in the module file and is loaded separately from the previous sections. *(Need to look into this mechanism more.)* Use the Data Block List and the Tag Struct List to identify the data stored here. Don't make assumptions.

This document will only cover the more complex fields. The others are straightforward (floats, integers, enums, etc.).

### `string_id` Fields

`string_id` fields are [Murmur3_32](https://en.wikipedia.org/wiki/MurmurHash) hashes of their strings. For example, 0x9B555AD2 = `"__default__"`. A useful online implementation of MurmurHash can be found [here](http://murmurhash.shorelabs.com/).

### Tag Block Fields (Size = 0x1C)

Tag block fields point to arrays of tag data structures. On disk, they don't actually store any info about where they point to. Use the Tag Struct List for that.

Offset | Type | Name | Description
--- | --- | --- | ---
0x00 | uint64 | `elements` | Runtime: pointer to the block containing the first struct.
0x08 | uint64 | `typeInfo` | Runtime: pointer to type information.
0x10 | int32 | `count` | Element count.
0x14 | int32 | `unknown14` | *(Unused?)*
0x18 | int32 | `unknown18` | *(Unused?)*

### Data Reference Fields (Size = 0x1C)

Data reference fields point to arbitrary data. On disk, they don't actually store any info about where they point to. Use the Data Reference List for that.

Offset | Type | Name | Description
--- | --- | --- | ---
0x00 | uint64 | `data` | Runtime: pointer to the block containing the first struct.
0x08 | uint64 | `typeInfo` | Runtime: pointer to type information.
0x10 | int32 | `unknown10` | *(Unused?)*
0x14 | int32 | `unknown14` | *(Unused?)*
0x18 | uint32 | `size` | Size of the referenced data in bytes.

### Tag Reference Fields (Size = 0x20)

Tag reference fields point to other tags. The Tag Reference List is the authoritative source of what these fields reference.

Offset | Type | Name | Description
--- | --- | --- | ---
0x00 | uint64 | `typeInfo` | Runtime: pointer to type information.
0x08 | uint32 | `nameLength` | Length of the tag's name in characters.
0x0C | int32 | `globalId` | Global ID of the tag.
0x10 | int64 | `assetId` | Asset ID of the tag.
0x18 | int32 | `groupTag` | Group tag (e.g. `proj`).
0x1C | int32 | `localHandle` | Runtime: the resolved local tag handle.

### Pageable Resource Fields (Size = 0x10)

Pageable resource fields point to resources. On disk, they don't actually store any info about where they point to. Use the Tag Struct List for that.

Offset | Type | Name | Description
--- | --- | --- | ---
0x00 | uint64 | `typeInfo` | Runtime: pointer to type information.
0x08 | uint32 | `handle` | Runtime: local resource handle.
0x0C | int32 | `unknownC` | *(Padding?)*

## Resource Data

The final section of the file (absent for most files) contains resource data. It needs to be looked into more.
