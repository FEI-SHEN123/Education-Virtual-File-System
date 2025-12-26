
## MUST (hard requirements)
1. C++ Client–Server architecture: interactive CLI client; requests over network; server parses and responds.
2. All data (papers/reviews/roles/versions) stored in server-side filesystem; client must not directly access local files.
3. Filesystem implementation: superblock, inode table, data blocks, free bitmap, multi-level directories, path resolution, create/delete, read/write.
4. Configurable-capacity LRU block cache + hit/miss/evict stats.
5. Backup feature: create/list/restore backups (admin only).
6. Multi-client concurrency + login authentication + permission checks.
7. Four roles and business mapping: author/reviewer/editor/admin commands and permissions.

## SHOULD (scoring / presentation)
1. Cover all technical requirements in implementation and report.
2. Be able to explain design optimizations and references to system/architecture ideas.
3. Provide a clear SOP and show collaboration workflow (GitHub).
4. Documentation should be comprehensive (more pages).
