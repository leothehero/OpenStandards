# BROFS - Bootloader Read-Only FileSystem

BROFS is the filesystem used by OpenBootLoader to load other filesystem drivers. It does not support directories, is limited to 64 files, and filenames are limited to 24 characters, making it in effect an organized pile of data with filenames on top.

BROFS's fixed-size file table occupies the first 1024 bytes (2 sectors) of a partition. On boot, this file table is loaded into memory and parsed.

Each inode is 32 bytes:

Bytes 1-2: Starting sector

Bytes 3-4: Size in bytes, rounded up to sectors.

Bytes 5-6: Preallocated space-- how big can it get?

Byte 7: flags, entry is only valid if >=1

Byte 8: Unused

Bytes 9-32: Filename (24 bytes)
