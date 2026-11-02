# Git Packfile Lab

A practical Git laboratory exploring object storage, packfiles, delta compression, garbage collection, and repository maintenance.

## Objectives

- Understand how Git stores objects.
- Explore the difference between loose objects and packed objects.
- Understand how Git creates and manages packfiles.
- Explore delta compression and delta chains.
- Learn how `git gc` and `git repack` maintain the object database.
- Understand how reflog expiration and unreachable object pruning affect recovery.

## 1. Git Object Storage

Git stores repository data as objects:

- **Blob:** Stores file contents.
- **Tree:** Stores directory structures and references to files and subdirectories.
- **Commit:** Stores commit metadata and references to a tree and parent commits.
- **Tag:** Stores annotated tag information.

Initially, Git stores objects individually as loose objects under `.git/objects`.

As the repository grows, Git can combine objects into packfiles to reduce storage overhead and improve efficiency.

## 2. Loose Objects vs Packed Objects

Loose objects:
- Stored individually in `.git/objects`.
- Each object is compressed.
- Useful for newly created objects before packing.

Packed objects:
- Multiple objects are stored together in a packfile.
- An accompanying index (`.idx`) enables efficient object lookup.
- Delta compression can reduce storage by recording differences between similar objects.

Useful commands:

```bash
git count-objects -v
git cat-file -t <object-id>
git cat-file -s <object-id>
git cat-file -p <object-id>
```

## 3. Git Garbage Collection

Git uses garbage collection to perform repository housekeeping.

```bash
git gc
```

This can:
- Pack loose objects.
- Repack existing objects.
- Remove unnecessary temporary files and obsolete packfiles.
- Prune eligible unreachable objects.

Git does not necessarily delete every unreachable object immediately.

## 4. Git Repack

`git repack` reorganizes objects into packfiles.

### Repack using existing packed objects

```bash
git repack -ad
```

- `-a`: Include all objects in the resulting pack.
- `-d`: Remove redundant packfiles after repacking.

### Force delta recompression

```bash
git repack -adf
```

- `-f`: Prevent reuse of existing deltas and recompute delta compression.

After repacking, inspect the object database:

```bash
git count-objects -v
Get-ChildItem .git/objects/pack
```

## 5. Delta Compression

Git can store similar objects efficiently by recording the differences between them instead of storing every version in full.

For example, when a log file changes slightly between commits, Git may store one full version and represent other versions as deltas.

Delta chains allow one delta-compressed object to depend on another object.

Inspect packfile contents:

```bash
git verify-pack -v <pack-index-file>
```

Important information includes:

- Object ID
- Object type
- Object size
- Compressed size
- Offset within the packfile
- Delta depth and base object, when applicable

Example from this lab:

```text
non delta: 11 objects
chain length = 1: 1 object
chain length = 2: 1 object
```

The pack contained 13 objects, with 2 delta-compressed objects.

## 6. Packfile Verification

Git provides a command to verify packfile integrity:

```bash
git verify-pack -v .git/objects/pack/<pack-file>.idx
```

A successful verification ends with:

```text
<pack-file>.pack: ok
```

The `.idx` file provides an index for locating objects inside the corresponding `.pack` file.

## 7. Reflog Expiration and Object Pruning

Reflogs record changes to Git references and can help recover commits after branch deletion or other reference changes.

Git has separate expiration settings:

| Configuration | Default |
|---|---|
| `gc.reflogExpire` | 90 days |
| `gc.reflogExpireUnreachable` | 30 days |
| `gc.pruneExpire` | 2 weeks ago |

Inspect configured values:

```bash
git config --show-origin --get-regexp "^gc\."
```

An empty result means no matching custom settings are explicitly configured. Git still applies its built-in defaults.

### Important distinction

- Reflog expiration removes old reference history.
- Object pruning removes eligible unreachable objects.
- Packfiles store Git objects efficiently; they are not guaranteed recovery backups.

An object can remain in the object database after its reflog entry expires. It may still be recoverable using its object ID until Git eventually prunes it.

## 8. Key Commands Reference

| Command | Purpose |
|---|---|
| `git count-objects -v` | Inspect loose and packed object counts |
| `git gc` | Perform repository housekeeping |
| `git repack -ad` | Repack all objects and remove redundant packs |
| `git repack -adf` | Repack without reusing existing deltas |
| `git verify-pack -v` | Inspect and verify a packfile |
| `git cat-file` | Inspect individual Git objects |
| `git fsck --full` | Check object database integrity |

## Conclusion

This lab demonstrates how Git stores repository objects, combines them into packfiles, uses delta compression to reduce storage requirements, and manages object lifecycles through garbage collection.

The experiments provide a practical understanding of Git's internal object storage and repository maintenance mechanisms.
