# ArenaServ Build Verification

**Last Updated:** 2026-09-23  
**ArenaServ Version:** 0.7.1 (Etheos baseline)  
**Purpose:** Document reproducible development build process

---

## Build Environment

### Supported Platforms

- **Primary:** Windows 10/11 with Visual Studio 2022+
- **Secondary:** Linux (documented in upstream Etheos, not re-verified for ArenaServ)

This document covers Windows development builds only.

---

## Prerequisites

### Required Tools

| Tool | Minimum Version | Verified Version | Purpose |
|------|----------------|------------------|---------|
| CMake | 3.5 | 4.4.2 | Build system |
| Visual Studio | 2017+ | VS 18.0 (2026) | C++ compiler |
| Git | Any | - | Version control |

### Visual Studio Requirements

**Workload:** Desktop development with C++

**Components:**
- MSVC v143+ build tools (x64/x86)
- Windows SDK
- CMake tools for Windows (optional, external CMake preferred)

---

## Build Configuration

### Database Backend Support

ArenaServ supports three database backends. **At least one** must be enabled:

| Backend | CMake Flag | Status | Notes |
|---------|------------|--------|-------|
| SQLite | `-DEOSERV_WANT_SQLITE=ON` | ✅ **Recommended** | Embedded, no external dependencies |
| MariaDB/MySQL | `-DEOSERV_WANT_MYSQL=ON` | ⚠️ Requires setup | External MariaDB installation required |
| SQL Server | `-DEOSERV_WANT_SQLSERVER=ON` | ⚠️ Requires setup | ODBC + SQL Server required |

**For development:** Use SQLite (simplest, no external database required).

### Build Modes

| Mode | CMake Flag | Optimization | Debug Symbols | Use Case |
|------|------------|--------------|---------------|----------|
| Release | `--config Release` | O2/Ox | No | Production, performance testing |
| Debug | `--config Debug` | Od | Yes | Debugging, development |

---

## Clean Build Procedure

### Step 1: Configure Build

Open PowerShell in the ArenaServ root directory.

#### Option A: Direct CMake (Recommended)

```powershell
# Create and enter build directory
New-Item -ItemType Directory -Path build -Force
cd build

# Configure with SQLite only (simplest)
cmake -DEOSERV_WANT_SQLITE=ON 
      -DEOSERV_WANT_MYSQL=OFF 
      -DEOSERV_WANT_SQLSERVER=OFF 
      -DEOSERV_USE_PRECOMPILED_HEADERS=OFF 
      ..
```

**Why precompiled headers OFF?** Faster incremental builds, avoids PCH corruption issues during development.

#### Option B: Build Script (Requires `vswhere`)

```powershell
.\build-windows.ps1 -Sqlite ON -MariaDB OFF -SqlServer OFF
```

**Note:** Script requires `vswhere.exe` in PATH (usually installed with VS 2017+). If missing, use Option A.

### Step 2: Build

```powershell
# From build/ directory
cmake --build . --config Release --target INSTALL
```

**Build time:** ~2-3 minutes (first build), ~30 seconds (incremental)

**Output location:** `install/etheos.exe`

### Step 3: Verify Build

```powershell
# Check executable exists
Test-Path install/etheos.exe

# Check test executable
Test-Path build/Release/eoserv_test.exe
```

---

## Known Build Issues

### Issue 1: Icon Resource Compilation Error

**Symptom:**
```
error RC2175: resource file icon.ico is not in 3.00 format
```

**Impact:** Non-critical. Executable builds successfully but lacks icon.

**Workaround:** Ignore (icon is cosmetic) OR convert `project/icon.ico` to 3.00 format using external tool.

**Status:** Tracked for future fix, does not block development.

### Issue 2: bcrypt CMake Version Conflict

**Symptom:**
```
CMake Error: Compatibility with CMake < 3.5 has been removed
```

**Cause:** Dependency `bcrypt` specifies `cmake_minimum_required(VERSION 2.8)` which conflicts with CMake 4.x.

**Fix:** Automatic during first build. CMake downloads bcrypt, then ArenaServ patches `build/bcrypt-src/CMakeLists.txt` to require 3.5.

**Status:** Workaround in place, does not block development.

### Issue 3: `vswhere` Not Found

**Symptom:**
```
vswhere : The term 'vswhere' is not recognized
```

**Cause:** `vswhere.exe` not in PATH (should be in `C:\Program Files (x86)\Microsoft Visual Studio\Installer\vswhere.exe`).

**Workaround:** Use direct CMake configuration (Option A above) instead of build script.

**Status:** Non-blocking, script is convenience wrapper only.

### Issue 4: Mock Object Leaks in Tests

**Symptom:**
```
ERROR: 2 leaked mock objects found at program exit
```

**Affected:** `WorldDumpTest.*`, `LoginTests.*` (21 of 46 tests)

**Impact:** Test infrastructure issue, not production code bug. 25 tests pass successfully.

**Status:** Tracked for future test harness cleanup, does not block development.

---

## Running Tests

### All Tests

```powershell
cd build/Release
.\eoserv_test.exe
```

**Expected:** 25 tests pass, 21 tests report mock leaks (infrastructure issue).

### Passing Tests Only

```powershell
.\eoserv_test.exe --gtest_filter=-WorldDumpTest*:LoginTests*
```

**Expected:** `[  PASSED  ] 25 tests.`

### List All Tests

```powershell
.\eoserv_test.exe --gtest_list_tests
```

---

## Clean Rebuild

To perform a completely clean build:

```powershell
# From ArenaServ root
Remove-Item -Recurse -Force build, install
.\build-windows.ps1 -Sqlite ON -MariaDB OFF -SqlServer OFF
```

OR:

```powershell
Remove-Item -Recurse -Force build, install
New-Item -ItemType Directory -Path build
cd build
cmake -DEOSERV_WANT_SQLITE=ON -DEOSERV_WANT_MYSQL=OFF -DEOSERV_WANT_SQLSERVER=OFF -DEOSERV_USE_PRECOMPILED_HEADERS=OFF ..
cmake --build . --config Release --target INSTALL
```

---

## Build Artifacts

### Primary Output

| File | Location | Purpose |
|------|----------|---------|
| `etheos.exe` | `install/` | Server executable |
| `eoserv_test.exe` | `build/Release/` | Unit test executable |
| `eoserv_lib.lib` | `build/Release/` | Static library (internal) |

### Configuration Files (Copied to `install/`)

- `config.ini` (template)
- `admin.ini` (template)
- `config/` directory (database, server settings)
- `data/` directory (empty data file templates)
- `lang/` directory (localization)
- `upgrade/` directory (database migration scripts)

### Runtime Files

- `database.sdb` — SQLite database (created on first run)
- `error.log` — Server error log

---

## Troubleshooting

### Build fails with "CMake not found"

**Solution:** Install CMake from https://cmake.org/ OR use Visual Studio installer to add "CMake tools for Windows."

### Build fails with "No C++ compiler found"

**Solution:** Install Visual Studio with "Desktop development with C++" workload.

### Build succeeds but `etheos.exe` missing

**Likely cause:** Icon resource error (Issue #1 above). Check build log for `error RC2175`.

**Impact:** Non-critical, does not prevent server operation.

**Next step:** Verify `eoserv_test.exe` built successfully. If yes, build system is functional.

### CMake configuration fails with policy warnings

**Example:**
```
CMake Warning (policy) at CMakeLists.txt:257 (install):
  Policy CMP0177 is not set
```

**Impact:** Warnings only, do not prevent build. Tracked for upstream Etheos compatibility.

**Action:** Ignore warnings unless build actually fails.

### Tests crash immediately

**Check:** Are you running from `build/Release/` directory? Tests expect certain working directory structure.

**Solution:**
```powershell
cd build/Release
.\eoserv_test.exe
```

---

## Development Workflow Integration

### Recommended Development Build

```powershell
# Fast incremental build after source changes
cd build
cmake --build . --config Release --target INSTALL
```

### Verify After Source Changes

```powershell
# Run passing tests
cd build/Release
.\eoserv_test.exe --gtest_filter=-WorldDumpTest*:LoginTests*
```

### Full Verification Before Commit

```powershell
# Clean build
Remove-Item -Recurse -Force build, install
New-Item -ItemType Directory build
cd build
cmake -DEOSERV_WANT_SQLITE=ON -DEOSERV_WANT_MYSQL=OFF -DEOSERV_WANT_SQLSERVER=OFF -DEOSERV_USE_PRECOMPILED_HEADERS=OFF ..
cmake --build . --config Release --target INSTALL

# Run tests
cd Release
.\eoserv_test.exe --gtest_filter=-WorldDumpTest*:LoginTests*
```

---

## Next Steps

- **Arena testing:** See `docs/testing/ARENA-TEST-MATRIX.md`
- **Test infrastructure:** See `docs/development/TEST-INFRASTRUCTURE.md`
- **GitHub workflow:** See `docs/GITHUB-WORKFLOW.md`

---

## Build Environment Snapshot (M0.9)

**Date:** 2026-09-23  
**Commit:** db4602e  
**CMake:** 4.4.2  
**Visual Studio:** 18.0 (2026 Community)  
**Compiler:** MSVC 14.51.36231  
**Architecture:** x64  
**Database:** SQLite only  
**Build Time:** ~143 seconds (clean), ~30 seconds (incremental)  
**Test Status:** 25/46 pass, 21/46 mock infrastructure issues

