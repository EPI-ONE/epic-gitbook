# Storage

Block contents are stored in files. Indices of where to find them in files are stored in a key-value database. Taking RocksDB as an example, we store the indices of blocks in two ColumnFamilies, and thus a searching query consists of two steps accordingly:

1. block hash --&gt; height + block offset in lvs + vertex offset in lvs
2. height --&gt;  block file position + vertex file position

First, we store block information and vertex information separately in two directories. The file position includes the subdirectory name, filename and an offset number of starting point of the level set where the block belongs in the file. The block location in file is obtained by adding level set offset and its offset in the level set. The same logic works on vertex information too. Moreover, the second table can be used to traverse the milestone chain.

**Note:** This requires blocks from the same level set can only be contained in one file. We use _max file size in bytes_ and _the number of level sets_ together to limit the size of each file.

The storage is manipulated by `block_store`. Only the the common part on each milestone chain branch goes into the storage. If there is only one branch, blocks before the current head height - 60 goes into the storage.

