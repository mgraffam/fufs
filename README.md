Landmark goals: Create a user-land tool to operate on high-level "container files"

*done	v0.0: Make main() with general options parsing needed, and some required stubs
*done	v0.1: Implement fufs_format to do the low-level formatting needed for the filesystem
*done	v0.2: Creating, and reading files
*done	v0.3: Add secure deletion 
---	v0.4: Implement directories   // works, but current implementation needs to be re-thought. Makes deleting dir entries and renaming files asinine. 
	v0.5: Add primitives for renaming files. 
	v0.6: Add full-file hashing to provide an error detection mechanism
	v0.7: Add primitive for appending data to an existing file-stream
	v0.8: Implement adding random chaff to the filesystem, and implement AES
	v0.9: Work the code into a nice state to use as a library for other tools. 
	v1.0: Code cleanup, organization, docs, github, etc. Possibly add support to delete/null
	         all blocks not used by specified keys. This provides a nuke capability, and the
	         ability to reclaim space used by chaff. 
	v1.1: Create a FUSE wrapper. Begin any work necessary to port to other FUSE-capable
	         platforms. 

Command line usage:

	--verbose		set verbose mode for output to stderr
	--register		sets the flag to register a new key .. shouldn't be needed unless debugging weird shit
	--format		Formats a filesystem
	-p / --password	        Declare a password (can be used multiple times, adding passphrases to the array each time)
				As of now, only passphrase[0] is used. This will be of more use later. 
	-f / --file arg		Specify the container file
	-i / --input arg	Specify the input file ("-" for stdin)
	-c / --create arg	Create a file named 'arg', reading from input
	-d / --delete arg	Delete a file named arg
	-r / --read arg		Read a file and print to stdout
	-m / --mkdir		Make a directory (linked to a given passphrase)
	-l / --list		Lists a directory (the directory contents)
	

Description of filesystem format:

Every FUFS partition begins with the 4-byte ID "FUFS", followed by a two-byte integer indicating the block size used to format the disk, followed by another two 
reserved bytes. At present, blocksize is hardcode by a #define, and the reserved bytes are unused. A some point, blocksize will be run-time definable when formatting
and read from this location and used otherwise. The reserved bytes will be used to encode which hash/cipher algorithms are used. Currently using GOST-R/Stribog for
pinning a passphrase/directory to a location on disk, and SHA3-512 for everything else. I intend to make this selectable amongst all 512 bit hashes, and may add
support for 256-bit hashes as well. 

After this 8-byte header, resides the allocation table. It is an array of bits used to represent the allocation state of every block of the disk, it 
is stored uncompressed, and unencrypted. The first N bits in the allocation table will always be set to 1's in order to preserve the header, and the allocation 
table itself. 

Following the allocation table, is the data portion of the disk. The data portion is treated as a number of blocks, each encrypted. When decrypted, the format is:

A two-byte integer representing the length of actual data in the block (to handle a short read at EOF on input), and an 8-byte integer representing the location of 
the next corresponding ciphertext block. "Head blocks" (first blocks of a file) contain a hash of the password used, and the rest of the remaining block is data. 
Subsequent blocks linked to by the location block will not have the hash. So, the first block of the file, and (in all probability) the last block of the file will
contain "short block" comparative to the others. This may get changed. I may keep the hash in every block, and use it in a running-tally form as I hash in more data.
hash(password) XOR hash(first block date) XOR hash (second block data) .. etc, and at each stage we expect the hash of the block to match. This would provide robust
error checking. 512 bits is overkill for an application like that, though. Maybe fold the 512 bit hashes onto themselves as I got and just keep 64-bits.

TODO: Whole bunches of shit. 

For starters, I use some float casting in places (eww!) with ceil to get some things done. Fucktarded, but was easy to do. Needs to get changed. Could be worse; 
I was using log2 to help make some binary calculations easier. At least I got rid of that. 

Error checking/reporting. Disk space full and normal stuff like that, plus lots of corner-cases need to get checked. It's difficult to write it though, because 
without working code its hard to come up with the error situations to test/debug the error checking. I gotta find a nice hex editor for Linux so that I can 
synthesize the errors.

