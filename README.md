========================================================================
                     CSE321 OPERATING SYSTEMS PROJECT
                      SIMPLE FILE SYSTEM (SimpleFS)
========================================================================

------------------------------------------------------------------------


------------------------------------------------------------------------
COMPILATION COMMANDS
------------------------------------------------------------------------
To compile the SimpleFS builder and adder programs, run the following GCC commands in the project directory:

  # Compile the File System Builder:
  gcc -Wall -Wextra -std=c11 simplefs_builder.c -o simplefs_builder

  # Compile the File System File Adder:
  gcc -Wall -Wextra -std=c11 simplefs_adder.c -o simplefs_adder


------------------------------------------------------------------------
EXECUTION EXAMPLES
------------------------------------------------------------------------
Step 1: Create/Format a new SimpleFS image file
  Command:
    ./simplefs_builder --image <image_name>
  Example:
    ./simplefs_builder --image disk.img
  Output:
    SimpleFS image created successfully: disk.img

Step 2: Add a file from host system to the SimpleFS image
  Command:
    ./simplefs_adder --input <image_name> --file <file_name>
  Example:
    ./simplefs_adder --input disk.img --file sample.txt
  Output:
    sample.txt added successfully to disk.img


------------------------------------------------------------------------
BRIEF DESCRIPTION OF THE IMPLEMENTATION
------------------------------------------------------------------------
SimpleFS is a lightweight, single-directory file system layout stored inside a single binary disk image file.

Layout Specifications:
  - Block Size: 4096 bytes (4 KB)
  - Total Disk Blocks: 64 blocks (Total Size: 256 KB)
  - Total Inodes: 32 inodes (Inodes 1 to 32)
  - Block Layout Breakdown:
      * Block 0: Superblock (holds magic number 0x53465331 and disk geometry)
      * Block 1: Inode Bitmap (tracks 32 inode allocation status)
      * Block 2: Data Bitmap (tracks 60 data block allocation status)
      * Block 3: Inode Table (contains 32 inode structures)
      * Blocks 4-63: Data Region (Block 4 is reserved for Root Directory data)

Implementation Details:

1. simplefs_builder.c:
   - Formats a 256 KB file initialized to zeros.
   - Writes the Superblock with magic number, block sizes, and structural locations.
   - Marks Inode 1 (Root Inode) in the Inode Bitmap and Data Block 4 (Root Data Region) in the Data Bitmap.
   - Configures Root Inode (Type 2/Directory, Size 128 bytes, 2 Link counts, Direct block [0] = 4).
   - Populates Root Directory with '.' (current directory, Inode 1) and '..' (parent directory, Inode 1) entries.

2. simplefs_adder.c:
   - Validates arguments, superblock integrity, and source file existence & size.
   - Checks if file already exists in the root directory to prevent duplicate file names.
   - Searches Inode Bitmap for a free inode (Indexes 1..31).
   - Searches Data Bitmap using a First-Fit strategy to allocate required data blocks.
   - Searches root directory block for a free entry slot (Inode 0).
   - Copies file content into allocated disk blocks in 4 KB chunks.
   - Initializes the new file's Inode (Type 1/File, link count 1, direct block pointers).
   - Flushes updated Inode Bitmap, Data Bitmap, new Inode, and Directory Entry back to the image file.
   - Increments Root Inode size by sizeof(dirent_t) (64 bytes).


------------------------------------------------------------------------
------------------------------------------------------------------------
KNOWN LIMITATIONS OR PROBLEMS
------------------------------------------------------------------------
1. Maximum File Size:
   Each inode only supports up to 3 direct block pointers (MAX_DIRECT_BLOCKS = 3).
   Therefore, the maximum file size supported is 3 * 4096 = 12,288 bytes (12 KB).
   Indirect blocks are not implemented.

2. Directory Entry Capacity:
   The root directory is allocated only 1 data block (4096 bytes). Since each directory
   entry (`dirent_t`) is 64 bytes, the directory can hold a maximum of 64 entries.
   With '.' and '..' taking 2 slots, at most 62 user files can be stored.

3. Flat Directory Structure:
   SimpleFS only supports a single root directory (`/`). Subdirectories cannot be created.

4. Deletion and Modification:
   File deletion, unlinking, renaming, or modifying existing file contents is not implemented.
   Files can only be appended to the root directory when space is available.

5. Fixed Disk Capacity:
   The file system layout is strictly fixed to 64 blocks (256 KB total).
========================================================================
