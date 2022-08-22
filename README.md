# teletext_archive
A format to deal with large amounts of teletext data and tools for it.


# Improved idea:

Referencing every individual packet is rather wastefull as a reference can easily have 32 bits. Since those references take up most of the space of an archive, it seems more efficient to reference whole pages instead. This way those 32 bits would not just reference a row (42 octets) but a whole page. Since the header row is most likely unique, there is little use in having it referenced.

## Implementation idea:

1. Variation on the .tta-idea: Header followed by packets for each dump.
2. Blocks of teletext packets referenced by header.

### Header idea:
* number of pages
** page/subpage number: 
** size/reference field


# Old idea below:

# Basic idea:
Normal teletext dumps in tta or t42 format do not benefit from leaving out redundant information. For example a daily dump of a teletext service will
store static pages in every dump even if they have not changed.

The basic idea is to have a set of files containing teletext packets. The dump itself will onyl contain links to theese packets. This will replace a 
42 byte octett with a 32 bit reference. Considering teletext services rarely exceed 50000 packets, this would allow for much more than 80000 dumps per 
archive.

!THIS IS WORK IN PROGRESS AND ESSENTIALLY A COLLECTION OF IDEAS!

## Chunks
Essentially the file would contain a series of chunks. Each chunk has a format like this:

| pos | size | name/comment|
|-----|------|-------------|
|  0  |    4 | size in octetts including header and footer |
|  4  |    4 | type  |
| 8---|variable | data
|end  |    4 | size in octetts including header, matches field at 0


## Chunk types

### Version chunk
Type: "VERS"
Data: "Version string"
This chunk should be at the start of the file and indicate the version

### Doku chunk (optional)
TYPE: DOKU
Data: UTF-8 text representing a human readable documentation of the file format

### Teletext packets
Type: "TXTP"
Data: teletext packets, each one 42 bytes.

### Pages
A page contains a pagenumber and subpage number as well as references to the data frames
Type: "PAG4"
Data:

| pos | size | name/comment|
+-----+------+-------------|
|   0 |    4 | page number |
| 4---| variable | refrences

The references contain unsigned pointers to within the file ignoring any file structure. This limits the file size to 4 Gigabytes.
If larger files are needed a possible extension would add a PAG5 header with longer pointers.

The page number would be 

### Capture
Contains a teletext capture.
Type: "CAP4"
Data:
| pos | size | name/comment |
+-----+------+--------------|
|   0 |    8 | timestamp of the capture in UNIX epochs

for each page
| pos | size | name/comment|
+-----+------+--------------
|   0 |    4 | page number like in PAG4
|   4 |    4 | pointer to the page

### Capture index
Type: "INDX"
Contains a file index, the last chunk should be of this type.
| pos | size | name/comment|
+-----+------+--------------
|   0 |    4 | pointer to the previous index
|   4 |    4 | 1 or more pointers to a CAP4 chunk


## Accessing a file
Accessing a file should start from the end. Since all chunks contain their length at the end it is easy t otraverse them from the end towards the start.
One an INDX-Chunk is found the pointers can be followed to to get to the pages of the capture or to the "next" capture. CAP4 chunks form a single linked
list from the end of the file to the start. This allows for adding new captures by appending to a file.
