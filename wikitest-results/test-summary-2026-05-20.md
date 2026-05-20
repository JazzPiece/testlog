# wikiGen Desktop — Test Results: 2026-05-20

## Overall Status: ⚠️ PARTIAL PASS

| Environment | Suite | Status | Result |
|---|---|---|---|
| A | Installation & Startup | ✅ PASS | 7/7 startup tests passed |
| B | Feature Testing | ⚠️ PARTIAL | 5 pass, 1 partial, 1 fail, 1 bug |
| C | Edge Cases & Stability | ⚠️ PARTIAL PASS | 5 pass/pass-with-warnings, 1 fail |

---

## Version Tested
**wikigen 1.1.0** (PyPI package `wikigen`)

---

## Installation Notes

The Linux desktop installer endpoint (`/api/track-install?platform=linux`) returns **HTTP 404** — the distribution route on the website is broken. The app was installed via **PyPI** (`pip install wikigen`) as a fallback, which is the functional Linux distribution path.

| Distribution channel | Status |
|---|---|
| `https://wiki-gen-library.vercel.app/api/track-install?platform=linux` | ❌ HTTP 404 |
| `https://github.com/JazzPiece/wikiGen-Library` | ❌ HTTP 404 (private/missing) |
| PyPI `pip install wikigen` | ✅ v1.1.0 available |
| Windows `.exe` installer | ❌ HTTP 404 |

---

## Environment A — Installation & Startup

All 7 tests passed. App starts cleanly on Python 3.12.

**Bugs (low/info):**
- **BUG-ENV-A-001 (low):** `--help` exits code 1 when piped (should be 0)
- **BUG-ENV-A-002 (low):** `config show` emits 5 duplicate keyring warnings without backend suppression
- **BUG-ENV-A-003 (info):** Config `output_dir` is global/shared — no per-environment isolation

---

## Environment B — Feature Testing

**Overall: partial** — core file crawling pipeline works; 3 bugs found.

| Feature | Result |
|---|---|
| Wiki creation — local dir crawl | ✅ partial (3 files crawled; hung at LLM step as expected with no key) |
| `config set` valid key | ✅ pass |
| `config set` invalid key | ❌ **bug** — unknown keys silently accepted |
| Invalid path handling | ✅ pass (exit 1, clear error) |
| Empty directory handling | ⚠️ partial (exit 1, generic error message) |
| `config show` | ✅ pass |
| `mcp` subcommand | 🐛 **bug** — zero output, not documented |

**Bugs:**
- **BUG-ENV-B-001 (medium):** `config set` accepts any unknown key without validation, writes it to config
- **BUG-ENV-B-002 (medium):** `wikigen mcp --help` exits 0 with zero bytes of output; `mcp` not in `--help`
- **BUG-ENV-B-003 (low):** Empty directory produces generic "Failed to fetch files" instead of descriptive message

---

## Environment C — Edge Cases & Stability

**Overall: partial pass** — no crashes or memory issues; 4 bugs found.

| Test | Result |
|---|---|
| 100-file large dataset crawl | ✅ PASS — all 100 files crawled, clean timeout at LLM step |
| File > `max_file_size` | ❌ FAIL — generic crash when all files skipped |
| Concurrent invocations (2 parallel) | ✅ PASS — no shared-state corruption |
| `config set` + run (persistence) | ✅ PASS — config change respected immediately |
| No-argument auto-CWD detection | ✅ PASS |
| Corrupted config recovery | ✅ PASS WITH WARNINGS — graceful fallback to defaults |
| Memory / process health | ✅ PASS — no segfaults or OOM kills |

**Bugs:**
- **EC-BUG-001 (medium):** Generic "Failed to fetch files" crash when all files exceed `max_file_size`
- **EC-BUG-002 (low):** wikigen writes `logs/` directory inside the analyzed repository (artifact pollution)
- **EC-BUG-003 (low):** Corrupted config warning emitted 4× per run instead of once
- **EC-BUG-004 (info):** Global config shared across all invocations; no per-project isolation

---

## Bug Summary

| ID | Severity | Title |
|---|---|---|
| BUG-ENV-A-001 | low | `--help` exits code 1 when piped |
| BUG-ENV-A-002 | low | 5× duplicate keyring warnings on `config show` |
| BUG-ENV-A-003 | info | Shared config output_dir — no per-environment isolation |
| BUG-ENV-B-001 | **medium** | `config set` accepts arbitrary unknown keys without validation |
| BUG-ENV-B-002 | **medium** | `mcp` subcommand silent, undocumented |
| BUG-ENV-B-003 | low | Generic "Failed to fetch files" for empty directory |
| EC-BUG-001 | **medium** | Generic crash when all files exceed `max_file_size` |
| EC-BUG-002 | low | `logs/` directory written inside analyzed repository |
| EC-BUG-003 | low | Corrupted config warning emitted multiple times per run |
| EC-BUG-004 | info | Global config — no per-project isolation |

**3 medium, 4 low, 3 info** — no high-severity or blocking bugs in the core pipeline.

---

## Test Artifacts

| File | Suite |
|---|---|
| `env-a/install-test-2026-05-20T121245Z.json` | Installation & Startup |
| `env-b/feature-test-2026-05-20T121245Z.json` | Feature Testing |
| `env-c/stability-test-2026-05-20T121245Z.json` | Edge Cases & Stability |
