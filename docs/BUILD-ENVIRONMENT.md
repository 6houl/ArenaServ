# ArenaServ Build Environment

**Status:** CONFIRMED  
**Last Updated:** 2026-09-22

## Overview

ArenaServ uses CMake for cross-platform builds supporting Windows and Linux.

## Requirements

### Core Build Tools

#### Windows
- **Visual Studio 2017-2022** with "Desktop Development with C++" workload
- **CMake** 3.5+ (3.31.6 recommended)
- **Git** (for dependency downloads)
- **vswhere** (included with Visual Studio Installer)

**Evidence:** `build-windows.ps1:51-87`, `README.md`

#### Linux
- **GCC** 4.8+ or **Clang** 3.4+
- **CMake** 2.6+
- **Git**
- **Make** or **Ninja**

**Evidence:** `build-linux.sh`, `CMakeLists.txt:82-93`

### Database Support

#### SQL Server (Windows)
- **Windows SDK** (for ODBC headers)
- Included with Visual Studio
- **ODBC** driver (odbccp32.dll)

**Evidence:** `CMakeLists.txt:217-222`

#### MariaDB/MySQL
- **MariaDB Connector/C** 3.4.8+
- Install location: `C:\Program Files (x86)\MariaDB\MariaDB Connector C\`
- Library: `libmariadb.dll`

**Evidence:** 
- `scripts/install-deps.ps1:90-106`
- `cmake/FindMariaDB.cmake`

#### SQLite3
- **SQLite3** amalgamation source
- On Windows: compiled directly into executable
- On Linux: system library required

**Evidence:**
- `scripts/install-deps.ps1:108-155`
- `CMakeLists.txt:224-235`
- `cmake/SourceFileList.cmake:263-267`

### External Dependencies

#### Bcrypt (Automatic)
- **Repository:** https://github.com/trusch/libbcrypt
- **Download:** Automatic via CMake
- **Purpose:** Password hashing

**Evidence:** `cmake/DownloadBcrypt.cmake`

#### GoogleTest (Automatic)
- **Repository:** https://github.com/google/googletest
- **Download:** Automatic via CMake
- **Purpose:** Unit testing

**Evidence:** `cmake/DownloadGoogleTest.cmake`

#### JSON Library (Manual)
- **Library:** nlohmann/json v3.9.1
- **Download:** Manual or via install script
- **Location:** `json/json.hpp`

**Evidence:** `scripts/install-deps.ps1:158-170`

## Build Configuration

### CMake Options

```cmake
# Database backend selection
-DEOSERV_WANT_MYSQL=ON|OFF      # MariaDB/MySQL support (default: ON)
-DEOSERV_WANT_SQLITE=ON|OFF     # SQLite3 support (default: ON)
-DEOSERV_WANT_SQLSERVER=ON|OFF  # SQL Server support (default: ON)

# Build optimization
-DEOSERV_USE_PRECOMPILED_HEADERS=ON|OFF  # Precompiled headers (default: ON)
-DEOSERV_USE_UNITY_BUILD=ON|OFF          # Unity build (default: ON)
-DEOSERV_USE_CLANG_MODULES=ON|OFF        # Clang modules (default: OFF)

# Development options
-DEOSERV_DEBUG_QUERIES=ON|OFF   # Print SQL queries to debug output
-DEOSERV_NO_DATA=ON|OFF         # Skip copying data files to build dir
-DEOSERV_OFFLINE=ON|OFF         # Build without internet (skip git downloads)
```

**Evidence:** `CMakeLists.txt:16-25`

### Compiler Flags

#### GCC/Clang
```bash
-std=c++17                # C++17 standard (or c++1y for older compilers)
-fwrapv                   # Signed overflow wrapping (required)
-fno-strict-aliasing      # Disable strict aliasing (required)
-Wall -Wextra             # Enable warnings
```

**Evidence:** `CMakeLists.txt:112-125`

#### MSVC
```
/utf-8                    # UTF-8 source files
_CRT_SECURE_NO_WARNINGS   # Disable secure CRT warnings
```

**Evidence:** `CMakeLists.txt:107-110`

### Platform Detection

```cmake
WIN32                     # Windows-specific code
UNIX or __linux__         # Linux-specific code
```

**Windows-specific files:**
- `src/eoserv_windows.h`
- `src/extra/ntservice.cpp/hpp`
- `project/winres.rc` (icon resource)

**Evidence:** `cmake/SourceFileList.cmake:258-262`

## Build Process

### Windows Build (PowerShell)

```powershell
# Full build with all databases
.\build-windows.ps1

# Debug build
.\build-windows.ps1 -Debug

# Select databases
.\build-windows.ps1 -SqlServer ON -MariaDB OFF -Sqlite ON

# Run tests after build
.\build-windows.ps1 -Test

# Clean build
.\build-windows.ps1 -Clean
```

**Script:** `build-windows.ps1`

#### Windows Build Steps
1. Check CMake availability
2. Locate Visual Studio via vswhere
3. Detect MariaDB in PATH (if enabled)
4. Create build directory
5. Generate Visual Studio solution
6. Build with MSBuild
7. Install to `install/` directory

**Evidence:** `build-windows.ps1:51-121`

### Linux Build (Bash)

```bash
# Full build
./build-linux.sh

# With options
./build-linux.sh -h           # Help
./build-linux.sh -d           # Debug mode
./build-linux.sh -t           # Run tests
./build-linux.sh -o           # Offline mode
```

**Script:** `build-linux.sh`

### Manual CMake Build

```bash
# Generate build files
mkdir build && cd build
cmake -G "Visual Studio 17 2022" -A Win32 ..

# Build
cmake --build . --config Release --target INSTALL

# Run tests
ctest -C Release
```

## Build Artifacts

### Output Structure

```
install/
├── etheos.exe (or etheos on Linux)      # Main server executable
├── test/
│   └── eoserv_test.exe                  # Test executable
├── LICENSE.txt
├── install.sql                          # MySQL/MariaDB schema
├── install_sqlserver.sql                # SQL Server schema
├── upgrade/                             # Database migration scripts
├── config/                              # Configuration templates
├── lang/                                # Language files
├── data/                                # Game data templates
└── config_local/                        # Local config directory
```

**Evidence:** `CMakeLists.txt:257-276`

### Build Directory Structure

```
build/
├── Release/ or Debug/              # Compiled binaries
├── bcrypt-src/                     # Downloaded bcrypt
├── bcrypt-build/                   # Built bcrypt library
├── googletest-src/                 # Downloaded GoogleTest
├── googletest-build/               # Built GoogleTest
├── CMakeCache.txt                  # CMake configuration
└── *.vcxproj (Windows) or Makefile (Linux)
```

## Dependency Installation

### Automated (Windows)

```powershell
# Requires administrator privileges
.\scripts\install-deps.ps1

# Options
-SkipCMake       # Don't install CMake
-SkipMariaDB     # Don't install MariaDB
-SkipSQLite      # Don't install SQLite
-CMakeVersion    # Specify CMake version (default: 3.31.6)
```

**Installs via Chocolatey:**
- CMake
- SQLite
- vswhere

**Direct downloads:**
- MariaDB Connector/C (MSI installer)
- SQLite amalgamation source
- nlohmann/json header

**Evidence:** `scripts/install-deps.ps1`

### Manual Installation

#### MariaDB Connector/C
```
URL: https://dlm.mariadb.com/4516899/Connectors/c/connector-c-3.4.8/mariadb-connector-c-3.4.8-win32.msi
Install: Run MSI, installs to C:\Program Files (x86)\MariaDB\
Add to PATH: {install}\lib
```

#### SQLite
```
Create directories:
  sqlite/src/
  sqlite/include/
  sqlite/bin/

Download: https://sqlite.org/2019/sqlite-amalgamation-3300100.zip
Extract: sqlite3.c → sqlite/src/
         sqlite3.h → sqlite/include/
```

#### nlohmann/json
```
Download: https://raw.githubusercontent.com/nlohmann/json/v3.9.1/single_include/nlohmann/json.hpp
Place: json/json.hpp
```

## Known Build Issues

### Issue: PowerShell Execution Policy
**Error:** Script cannot be loaded (not digitally signed)
**Solution:** Run with bypass: `powershell -ExecutionPolicy Bypass -File .\build-windows.ps1`
**Evidence:** Session testing

### Issue: vswhere Not in PATH
**Error:** 'vswhere' is not recognized
**Solution:** vswhere is at `${env:ProgramFiles(x86)}\Microsoft Visual Studio\Installer\vswhere.exe`
**Fix:** install-deps.ps1 installs it, or add to PATH
**Evidence:** `build-windows.ps1:65-87`

### Issue: Icon Resource Invalid
**Error:** `RC2175: resource file icon.ico is not in 3.00 format`
**Cause:** Icon file is git-lfs pointer, not actual icon
**Solution:** 
1. Use git-lfs: `git lfs install && git lfs pull`
2. Or comment out icon line in `project/winres.rc`
**Evidence:** Session testing, `README.md:30-38`

### Issue: bcrypt CMakeLists.txt Version
**Error:** `Compatibility with CMake < 3.5 has been removed`
**Cause:** Downloaded bcrypt has old CMake minimum version
**Solution:** Auto-patched during build or update `build/bcrypt-src/CMakeLists.txt`
**Evidence:** Session testing

## Build Performance

### Unity Build
- **Enabled by default:** Yes
- **Effect:** Faster compilation (fewer translation units)
- **Files:** `tu/*.cpp` compile multiple sources
- **Trade-off:** Less granular dependency tracking

**Evidence:** `CMakeLists.txt:56-61`

### Precompiled Headers
- **Enabled by default:** Yes (except Windows default: OFF)
- **Header:** Generated from `src/stdafx.h` or auto-generated
- **Effect:** Faster recompilation
- **Windows:** Disabled by default due to compatibility

**Evidence:** `CMakeLists.txt:288-308`

## Testing

### Unit Tests

```bash
# Run all tests
cd build
ctest -C Release

# Or run directly
./install/test/eoserv_test.exe (Windows)
./install/test/eoserv_test (Linux)
```

**Test files:**
- `src/test/config_test.cpp`
- `src/test/database_test.cpp`
- `src/test/worlddump_test.cpp`
- `src/test/handlers/Login_test.cpp`
- `src/test/util/semaphore_test.cpp`
- `src/test/util/threadpool_test.cpp`

**Evidence:** `cmake/SourceFileList.cmake:299-306`

### Integration Tests (Linux only)

Requires EOBot from EndlessClient project.

```bash
cd deploy
./ci-test.sh --self-contained --use-local --botdir path/to/eobot
```

**Tests:** `src/test/integration/` (test scripts)

**Evidence:** `README.md:138-171`

## Continuous Integration

### GitHub Actions

**Workflows:**
- `.github/workflows/build.yaml` - Build verification
- `.github/workflows/ci.yaml` - Integration tests and deployment

**Evidence:** `.github/workflows/`

## Compiler Support

### Supported Compilers

| Compiler | Minimum Version | Status |
|----------|----------------|--------|
| MSVC | 14.0 (VS 2015) | ✅ CONFIRMED |
| GCC | 4.8 | ✅ CONFIRMED |
| Clang | 3.4 | ✅ CONFIRMED |

**Evidence:** `CMakeLists.txt:82-103`

### C++ Standard

- **Target:** C++17
- **Fallback:** C++14 (c++1y) for GCC < 5.0, Clang < 3.5
- **Required Features:**
  - Lambda expressions
  - Auto type deduction
  - Range-based for loops
  - nullptr
  - Override/final
  - Constexpr

**Evidence:** `CMakeLists.txt:112-125`

## Development Environment Recommendations

### IDE Support

- **Visual Studio 2022** (Windows) - Native CMake support
- **Visual Studio Code** - CMake Tools extension
- **CLion** - Native CMake support
- **Vim/Emacs** - LSP with clangd

### Recommended Extensions (VS Code)
- C/C++ (ms-vscode.cpptools)
- CMake (twxs.cmake)
- CMake Tools (ms-vscode.cmake-tools)

**Evidence:** `README.md:112-118`

---

**Summary:** ArenaServ has a well-configured cross-platform build system. Windows builds via PowerShell + Visual Studio, Linux via bash + GCC/Clang. CMake handles all dependencies automatically except data files. Testing framework included.
