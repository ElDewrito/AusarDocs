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
0x24 | int32 | `numCompressedBlocks` | Number of compressed blocks in the module
0x28 | uint8[0x10] | `unknown28` |

## File List

The file list immediately follows the header. It stores metadata for each file, including where the file is located in the module and how it is compressed.
Files are compressed using zlib.

### File Entry Structure (Size: 0x58)

Offset | Type | Name | Description
--- | --- | --- | ---
0x00 | int32 | `nameOffset` | Offset of the filename in the filename data
0x04 | int32 | `parentFileIndex` | Used with resources to point back to the parent file. -1 = none
0x08 | int32 | `unknown8` |
0x0C | int32 | `unknownC` |
0x10 | int32 | `compressedBlockCount` | The number of compressed blocks that make up the file. If this is 0, then just use totalCompressedSize and totalUncompressedSize as a block.
0x14 | int32 | `firstCompressedBlockIndex` | The index of the first compressed block in the file.
0x18 | uint64 | `compressedOffset` | The offset of the first compressed block in the file, relative to the start of the compressed data area in the module. This area follows the compressed block list.
0x20 | uint32 | `totalCompressedSize` | The total size of compressed data.
0x24 | uint32 | `totalUncompressedSize` | The total size of the data after it is uncompressed. If this is 0, then the file is empty.
0x28 | uint8 | `unknown28` |
0x29 | uint8 | `unknown29` |
0x2A | uint8 | `unknown2A` |
0x2B | uint8 | `unknown2B` |
0x2C | int32 | `unknown2C` |
0x30 | int32 | `unknown30` |
0x34 | int32 | `unknown34` |
0x38 | uint64 | `unknown38` |
0x40 | int32 | `groupTag` | If the file is a tag, this holds the group tag of the file (e.g. `bipd`).
0x44 | uint32 | `uncompressedHeaderSize` | The size of the file's uncompressed header.
0x48 | uint32 | `uncompressedTagDataSize` | The size of the file's uncompressed tag data.
0x4C | uint32 | `uncompressedResourceDataSize` | The size of the file's uncompressed resource data.
0x50 | int16 | `unknown50` |
0x52 | int16 | `unknown52` |
0x54 | int32 | `unknown54` |

## Filename Data

The filename data immediately follows the file list. It is simply a conglomeration of null-terminated ASCII strings. Offsets into this are stored in the `nameOffset` field of each file entry.

## Resource List

This resource list immediately follows the filename data. It is a list of 4-byte indices which point to files in the File List that contain resource data.

## Compressed Blocks

The compressed block list immediately follows the root tag list. Each file is split into multiple blocks which are compressed individually, and this lists information about each block.

### Compressed Block Structure (Size: 0x20)

Offset | Type | Name | Description
--- | --- | --- | ---
0x00 | int64 | `unknown0` |
0x08 | uint32 | `compressedOffset` | The offset of this block relative to the start of the file's compressed data.
0x0C | uint32 | `compressedSize` | The size of the compressed block.
0x10 | uint32 | `uncompressedOffset` | The offset of the block's uncompressed data in the uncompressed file.
0x14 | uint32 | `uncompressedSize` | The size of the block's data after it is uncompressed.
0x18 | int32 | `unknown18` |
0x1C | int32 | `unknown1C` |

## Compressed Data

Following the compressed block list is the compressed data area of the module. This is where all compressed data lies.

## Steps to Extract a File

These are the steps you need to take to fully extract a file from a module:

1. Read the header, file list, filename data, root tag list, and compressed block list.
2. The file pointer should now point to the start of the compressed data area. Save this value (we'll call it `compressedDataOffset`).
3. Go to the file's entry in the file list.
4. If you need the file's name, retrieve it by reading the ASCII string at the file's `nameOffset` in the filename data.
5. If the file's `totalUncompressedSize` is 0, then it is empty. Stop.
6. Compute the offset of the first compressed block in the file as `firstCompressedBlockOffset = compressedDataOffset + file.compressedOffset`).
7. Check the file's `compressedBlockCount`.
  * If it's 0, then define a block using the file's `totalCompressedSize` and `totalUncompressedSize`.
  * Otherwise, get its compressed blocks out of the compressed blocks list using the file's `firstCompressedBlockIndex` and `compressedBlockCount`.
8. For each compressed block, seek to the offset `firstCompressedBlockOffset + block.compressedOffset` in the module file. Read `block.compressedSize` bytes and uncompress them using zlib.
9. Copy each uncompressed block to the output buffer using the block's `uncompressedOffset` and `uncompressedSize`.

You can see a sample implementation of this [here](https://github.com/Shockfire/FiveTool/blob/master/ModuleExtractor/main.cpp).
