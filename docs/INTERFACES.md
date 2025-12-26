cat > docs/INTERFACES.md <<'EOF'
# INTERFACES (Day1 Freeze)

Version: v1.0
Last updated: 2025-12-26

---

## 0. Conventions

### Error codes
All APIs return:
- `0` for OK
- negative for errors:

| Code | Name        | Meaning |
|------|-------------|---------|
| -1   | EPERM       | permission denied / not allowed |
| -2   | ENOENT      | path or entry not found |
| -3   | EEXIST      | already exists |
| -4   | ENOTDIR     | expected directory |
| -5   | EISDIR      | is a directory |
| -6   | ENOTEMPTY   | directory not empty |
| -7   | EINVAL      | invalid argument / invalid path |
| -8   | ENOSPC      | no space left (blocks/inodes) |
| -9   | EIO         | I/O error / corrupted data |

### Path rules (VFS paths)
- Absolute path only, must start with `/`.
- Normalize:
  - collapse multiple slashes: `//` -> `/`
  - ignore `.` segments
  - `..` policy: **REJECT** for v1.0 (return `-EINVAL`) to keep implementation simple.
- `/` is the root directory.

---

## 1. IBlockDevice (Module A <-> Module B / Storage)

### Purpose
Provide block-level I/O for VFS. VFS MUST NOT directly read/write OS files except via an implementation of IBlockDevice.

### Semantics
- Fixed block size for a device.
- `ReadBlock/WriteBlock` must read/write **exactly one full block**.
- `block_id` range: `[0, NumBlocks()-1]`.
- Block 0 is reserved for SuperBlock in our VFS format.

### Signature (C++ style)
```cpp
#pragma once
#include <cstdint>
#include <cstddef>

struct IBlockDevice {
  virtual ~IBlockDevice() = default;

  virtual uint32_t BlockSize() const = 0;      // e.g. 4096
  virtual uint32_t NumBlocks() const = 0;      // total blocks on "disk"

  // Read exactly one block into out[BlockSize()].
  // Return true on success, false on I/O error or out-of-range.
  virtual bool ReadBlock(uint32_t block_id, void* out) = 0;

  // Write exactly one block from data[BlockSize()].
  // Return true on success, false on I/O error or out-of-range.
  virtual bool WriteBlock(uint32_t block_id, const void* data) = 0;

  // Persist any buffered writes (if any).
  virtual bool Flush() = 0;
};## 2. VfsAPI (Server/Business -> VFS)

Purpose:
Server calls VfsAPI to access the virtual filesystem stored inside the block device.

Mount lifecycle:
- Mkfs(dev): formats a blank VFS on the device (destroys old contents).
- Mount(dev): loads/validates the VFS and returns a ready VFS instance.

Required operations (v1.0):

```cpp
#pragma once
#include <cstdint>
#include <memory>
#include <string>
#include <vector>

struct IBlockDevice;

class Vfs {
public:
  // Format
  static bool Mkfs(std::shared_ptr<IBlockDevice> dev);

  // Mount existing filesystem
  static std::unique_ptr<Vfs> Mount(std::shared_ptr<IBlockDevice> dev);

  // Directory ops
  int Mkdir(const std::string& path);
  int Rmdir(const std::string& path);
  int ListDir(const std::string& path, std::vector<std::string>* out_names);

  // File ops
  int CreateFile(const std::string& path);
  int Unlink(const std::string& path);

  // Read/write by offset
  int ReadFile(const std::string& path, uint64_t off, uint32_t n, std::string* out);
  int WriteFile(const std::string& path, uint64_t off, const std::string& data);

  // Optional
  int Truncate(const std::string& path, uint64_t new_size);
};
