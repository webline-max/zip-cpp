# Webline_Zip
---

Small C++ zlib/minizip wrapper for ZIP archive manipulation (based on minizip & zlib 1.3.1)

What it does?

- Opens and reads ZIP archives (including ZIP64)
- Extracts files with optional password decryption
- Creates new ZIP archives with compression
- Supports streaming I/O via custom callbacks
- Handles large files (>4GB) transparently
- Includes full zlib 1.3.1 deflate/inflate engine

Requirements

None except a C++ compiler.

Uses these stdlib headers:

| Header     | Purpose                                    |
|------------|--------------------------------------------|
| stdio.h    | for FILE operations fallback               |
| stdlib.h   | for malloc/free                            |
| string.h   | for memcpy/memset                          |
| stdint.h   | for uint64_t etc.                          |
| errno.h    | for error codes                            |
| fcntl.h    | for open flags                             |
| time.h     | for DOS date conversion                    |

How to build

1. Include `zip.cpp` in your project
2. Link against the compiled object or include as header-only if you define implementation macros
3. No external dependencies - zlib is bundled

```cpp
#include "zip.cpp"
```

How to use

### Reading a ZIP file

```cpp
unzFile zf = unzOpen("archive.zip");
if (!zf) {
    printf("Failed to open archive\n");
    return;
}

// Iterate through files
if (unzGoToFirstFile(zf) == UNZ_OK) {
    do {
        char filename[256];
        unz_file_info info;
        
        unzGetCurrentFileInfo(zf, &info, filename, sizeof(filename), 
                              NULL, 0, NULL, 0);
        printf("File: %s (%lu bytes)\n", filename, info.uncompressed_size);
        
        // Extract current file
        if (unzOpenCurrentFile(zf) == UNZ_OK) {
            unsigned char buf[4096];
            int bytesRead;
            while ((bytesRead = unzReadCurrentFile(zf, buf, sizeof(buf))) > 0) {
                // Process extracted data
                fwrite(buf, 1, bytesRead, stdout);
            }
            unzCloseCurrentFile(zf);
        }
    } while (unzGoToNextFile(zf) == UNZ_OK);
}

unzClose(zf);
```

### Creating a ZIP file

```cpp
zipFile zf = zipOpen("output.zip", APPEND_STATUS_CREATE);
if (!zf) {
    printf("Failed to create archive\n");
    return;
}

zip_fileinfo zi = {0};
// Set timestamp if needed
// zi.tmz_date.tm_year = 2026; etc.

if (zipOpenNewFileInZip(zf, "hello.txt", &zi, 
                        NULL, 0, NULL, 0, 
                        "Comment", Z_DEFLATED, Z_DEFAULT_COMPRESSION) == ZIP_OK) {
    const char* data = "Hello, World!";
    zipWriteInFileInZip(zf, data, strlen(data));
    zipCloseFileInZip(zf);
}

zipClose(zf, "Global comment");
```

API Functions:

Archive reading (unzip):

| Function | Description |
|----------|-------------|
| unzOpen() | Open ZIP archive for reading |
| unzOpen64() | Open ZIP64 archive |
| unzClose() | Close archive |
| unzGoToFirstFile() | Move to first entry |
| unzGoToNextFile() | Move to next entry |
| unzLocateFile() | Find file by name |
| unzGetCurrentFileInfo() | Get metadata of current file |
| unzOpenCurrentFile() | Begin extraction |
| unzReadCurrentFile() | Read decompressed data |
| unzCloseCurrentFile() | End extraction |

Archive writing (zip):

| Function | Description |
|----------|-------------|
| zipOpen() | Create/open archive for writing |
| zipOpenNewFileInZip() | Add new file entry |
| zipWriteInFileInZip() | Write compressed data |
| zipCloseFileInZip() | Finalize current entry |
| zipClose() | Finalize and close archive |

Utility / Advanced:

| Function | Description |
|----------|-------------|
| fill_fopen_filefunc() | Setup default stdio I/O callbacks |
| fill_fopen64_filefunc() | Setup 64-bit I/O callbacks |
| compress() / uncompress() | Raw zlib memory compression |
| crc32() / adler32() | Checksum helpers |

---
