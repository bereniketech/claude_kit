---
name: content-hash-cache-pattern
description: Cache expensive file processing results using SHA-256 content hashes — path-independent, auto-invalidating, with service layer separation.
---

# Content-Hash Cache Pattern

Cache expensive file processing results (PDF parsing, text extraction, image analysis) using SHA-256 content hashes as cache keys. Unlike path-based caching, this survives file moves/renames and auto-invalidates when content changes.

---

## 1. Compute the Cache Key

Hash file content, not the file path:

```python
import hashlib
from pathlib import Path

_HASH_CHUNK_SIZE = 65536  # 64 KB chunks

def compute_file_hash(path: Path) -> str:
    sha256 = hashlib.sha256()
    with open(path, "rb") as f:
        while chunk := f.read(_HASH_CHUNK_SIZE):
            sha256.update(chunk)
    return sha256.hexdigest()
```

**Rule:** Always chunk large files — never load the entire file into memory just to hash it.

---

## 2. Store and Retrieve Cache Entries

Name each entry `{hash}.json` for O(1) lookup with no index file required:

```python
def write_cache(cache_dir: Path, file_hash: str, data: dict) -> None:
    cache_dir.mkdir(parents=True, exist_ok=True)
    (cache_dir / f"{file_hash}.json").write_text(
        json.dumps(data, ensure_ascii=False), encoding="utf-8"
    )

def read_cache(cache_dir: Path, file_hash: str) -> dict | None:
    cache_file = cache_dir / f"{file_hash}.json"
    if not cache_file.is_file():
        return None
    try:
        return json.loads(cache_file.read_text(encoding="utf-8"))
    except (json.JSONDecodeError, ValueError, KeyError):
        return None  # Corruption = cache miss, never crash
```

**Rule:** Treat any deserialization error as a cache miss — graceful degradation is mandatory.

---

## 3. Wrap with a Service Layer

Keep the processing function pure. Add caching as a separate wrapper — never mix the two responsibilities:

```python
def process_with_cache(
    file_path: Path,
    *,
    cache_enabled: bool = True,
    cache_dir: Path = Path(".cache"),
):
    if not cache_enabled:
        return process_file(file_path)  # pure function, no cache knowledge

    file_hash = compute_file_hash(file_path)
    cached = read_cache(cache_dir, file_hash)
    if cached is not None:
        return cached

    result = process_file(file_path)
    write_cache(cache_dir, file_hash, result)
    return result
```

**Rule:** The core processing function must have zero knowledge of caching. SRP violation = future maintenance pain.

---

## 4. Cache Invalidation

Content hashes self-invalidate: when file bytes change, the hash changes, producing a new cache key. No explicit invalidation logic is needed.

Use cases where this pattern fits:
- File processing pipelines (PDF, OCR, image analysis)
- CLI tools with `--cache` / `--no-cache` flags
- Batch jobs where the same files reappear across runs

Use cases where it does NOT fit:
- Data that must always be fresh (real-time feeds)
- Results that depend on parameters beyond file content (different extraction configs require a composite key)
- Extremely large cache entries (consider streaming instead)

---

## 5. Anti-Patterns to Avoid

- **Path-based keys** — breaks on file rename/move
- **Cache logic inside the processing function** — SRP violation
- **`dataclasses.asdict()` on nested frozen dataclasses** — use manual serialization
- **No corruption handling** — always wrap deserialization in try/except

---
