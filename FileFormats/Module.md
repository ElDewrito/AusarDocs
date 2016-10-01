# Module File Format

.module files replace the cache map system used by older versions of the Blam engine.
Under this new system, tags are linked together much more loosely and multiple modules can be loaded on top of each other (referred to as a `modulestack`).

Module files are primarily an archive format. While most of the files in them are generally tags or resources, they can contain non-tag files as well (e.g. `loadmanifest`).

This document covers version **27** (0x1B) of the module format.

## File Header

The file header contains version information about the module file alongside information about how the other sections of the file are laid out.

### File Header Structure (Size: 0x38)

Offset | Type | Name | Description
--- | --- | --- | ---
0x00 | int32 | `magic` | `'dhom'`
0x04 | int32 | `version` | 27 (0x1B)
0x08 | int32 | `unknown8` |
0x0C | int32 | `unknownC` |
0x10 | int32 | `numFiles` | Number of files stored in the module
0x14 | int32 | `unknown14` |
0x18 | int32 | `unknown18` |
0x1C | int32 | `fileNamesSize` | Size of the filename data in the module
0x20 | int32 | `numResources` | Number of resource files in the module
0x24 | int32 | `numFileBlocks` | Number of file blocks in the module
0x28 | uint64 | `unknown28` |
0x30 | uint64 | `checksum` | Murmur3_x64_128 of the header (set this field to 0 first), file list, resource list, and block list

## File List

The file list immediately follows the header. It stores metadata for each file, including where the file is located in the module and how it is stored.

### File Entry Structure (Size: 0x58)

Offset | Type | Name | Description
--- | --- | --- | ---
0x00 | int32 | `nameOffset` | Offset of the filename in the filename data
0x04 | int32 | `parentFileIndex` | Used with resources to point back to the parent file. -1 = none
0x08 | int32 | `resourceCount` | Number of resources this file owns
0x0C | int32 | `firstResourceIndex` | Index of the first resource in the module's resource list that this file owns (see the Resource List section below).
0x10 | int32 | `blockCount` | The number of blocks that make up the file. Only valid if the `HasBlocks` flag is set (see below).
0x14 | int32 | `firstBlockIndex` | The index of the first block in the file. Only valid if the `HasBlocks` flag is set (see below).
0x18 | uint64 | `dataOffset` | The offset of the first block in the file, relative to the start of the data area in the module. This area follows the block list.
0x20 | uint32 | `totalCompressedSize` | The total size of compressed data.
0x24 | uint32 | `totalUncompressedSize` | The total size of the data after it is uncompressed. If this is 0, then the file is empty.
0x28 | uint8 | `headerAlignment` | Power of 2 to align the header buffer to (e.g. 4 = align to a multiple of 16 bytes).
0x29 | uint8 | `tagDataAlignment` | Power of 2 to align the tag data buffer to.
0x2A | uint8 | `resourceDataAlignment` | Power of 2 to align the resource data buffer to.
0x2B | uint8 | `flags` | See "File Entry Flags" below.
0x2C | int32 | `globalTagId` | The global tag ID (-1 if not a tag).
0x30 | int64 | `assetId` | The asset ID (-1 if not a tag).
0x38 | uint64 | `assetChecksum` | Murmur3_x64_128 hash of (what appears to be) the original file that this file was built from. This is not always the same thing as the file stored in the module. Only verified if the `HasBlocks` flag is not set.
0x40 | int32 | `groupTag` | If the file is a tag, this holds the group tag of the file (e.g. `bipd`).
0x44 | uint32 | `uncompressedHeaderSize` | The size of the file's uncompressed header.
0x48 | uint32 | `uncompressedTagDataSize` | The size of the file's uncompressed tag data.
0x4C | uint32 | `uncompressedResourceDataSize` | The size of the file's uncompressed resource data.
0x50 | int16 | `headerBlockCount` | Number of blocks that make up the file's header.
0x52 | int16 | `tagDataBlockCount` | Number of blocks that make up the file's tag data.
0x54 | int16 | `resourceBlockCount` | Number of blocks that make up the file's resource data.
0x56 | int16 | `padding56` | _(Padding)_

### File Entry Flags

Each entry has a flags field which describes how the file is stored in the module. See the value at offset 0x2B.

Bit | Name | Description
--- | --- | ---
0 | `Compressed` | The file uses compression.
1 | `HasBlocks` | The `blockCount` and `firstBlockIndex` fields are valid. If this is not set, then the file only has one block which can be inferred from the fields in the struct.
2 | `RawFile` | The file is not split into header/tag data/resource sections and should be read in its entirety.

## Filename Data

The filename data immediately follows the file list. It is simply a conglomeration of null-terminated ASCII strings. Offsets into this are stored in the `nameOffset` field of each file entry.

## Resource List

This resource list immediately follows the filename data. It is a list of 4-byte indices which point to files in the File List that contain resource data.

## Block List

The block list immediately follows the root tag list. Each file is split into multiple blocks which can be compressed individually, and this lists information about each block.

### Block Definition Structure (Size: 0x20)

Offset | Type | Name | Description
--- | --- | --- | ---
0x00 | uint64 | `checksum` | Murmur3_x64_128 hash of the uncompressed block data (the upper 64 bits are discarded)
0x08 | uint32 | `compressedOffset` | The offset of this block relative to the start of the file's compressed data.
0x0C | uint32 | `compressedSize` | The size of the compressed data.
0x10 | uint32 | `uncompressedOffset` | The offset of the block's uncompressed data in the uncompressed file.
0x14 | uint32 | `uncompressedSize` | The size of the block's data after it is uncompressed.
0x18 | bool | `compressed` | `true` if the block data is compressed using ZLib, `false` if it should just be read verbatim
0x1C | int32 | `padding1C` | _(Padding)_

## File Data

Following the block list is the data area of the module. This is where all file data lies.

## Steps to Extract a File

These are the steps you need to take to fully extract a file from a module:

1. Read the header, file list, filename data, root tag list, and block list.
2. The file pointer should now point to the start of the data area. Save this value (we'll call it `dataOffset`).
3. Go to the file's entry in the file list.
4. If you need the file's name, retrieve it by reading the ASCII string at the file's `nameOffset` in the filename data.
5. If the file's `totalUncompressedSize` is 0, then it is empty. Stop.
6. Compute the offset of the first block in the file as `firstBlockOffset = dataOffset + file.dataOffset`).
7. Check the file's `HasBlocks` flag.
  * If 0, then define a block using the relevant fields in the file struct. Make sure to check the `Compressed` flag!
  * If 1, then get its blocks out of the block list using the file's `firstBlockIndex` and `blockCount`.
8. For each block, seek to the offset `firstBlockOffset + block.compressedOffset` in the module file. Read `block.compressedSize` bytes.
  * If `block.compressed` is `true`, uncompress them using zlib.
9. Copy each uncompressed block to the output buffer using the block's `uncompressedOffset` and `uncompressedSize`.

You can see a sample implementation of this [here](https://github.com/Shockfire/FiveTool/blob/master/ModuleExtractor/main.cpp).
