# GitHub Actions Workflows

This directory contains CI/CD workflows for building and releasing libzmq native libraries.

## Workflows

### 1. build.yml - Main Build Pipeline
**Purpose:** Build libzmq for all supported platforms

**Triggers:**
- Push to `main` or `develop` branches
- Tag push (`v*`)
- Pull request to `main`
- Manual dispatch (with optional version override)

**Jobs:**
- `read-versions`: Parse version information from VERSION file
- `build-windows`: Build Windows x64 binaries
- `build-linux`: Build Linux x64 binaries
- `build-macos-intel`: Build macOS x64 (Intel) binaries
- `build-macos-arm64`: Build macOS ARM64 (Apple Silicon) binaries
- `verify`: Download all artifacts, generate checksums, and create summary

**Artifacts:**
- `libzmq-windows-x64`: Windows x64 build output
- `libzmq-linux-x64`: Linux x64 build output
- `libzmq-macos-x64`: macOS x64 build output
- `libzmq-macos-arm64`: macOS ARM64 build output
- `checksums`: SHA256 checksums for all binaries

**Retention:** 30 days

### 2. release.yml - Release Pipeline
**Purpose:** Create GitHub releases with packaged binaries

**Triggers:**
- Tag push (`v*`)
- Manual dispatch (with tag name input)

**Process:**
1. Calls `build.yml` workflow to build all platforms
2. Downloads all artifacts
3. Packages each platform into `.tar.gz` archives
4. Generates SHA256 checksums
5. Creates GitHub release with:
   - Release notes (auto-generated)
   - Platform packages
   - Checksums
   - Build metadata

**Permissions:** Requires `contents: write` for release creation

### 3. pr-check.yml - Pull Request Validation
**Purpose:** Quick validation for pull requests

**Triggers:**
- Pull request to `main` or `develop`
- Only when build scripts, VERSION file, or workflows are modified

**Jobs:**
- `check-versions`: Validate VERSION file format
- `build-sample`: Build sample platforms (Windows and Linux only)
- `pr-summary`: Generate PR check summary

**Note:** This is a lightweight check. Full multi-platform builds run after merge.

## Status Badges

Add these badges to your README.md:

```markdown
[![Build Status](https://github.com/YOUR_USERNAME/libzmq-native/actions/workflows/build.yml/badge.svg)](https://github.com/YOUR_USERNAME/libzmq-native/actions/workflows/build.yml)
[![Release](https://github.com/YOUR_USERNAME/libzmq-native/actions/workflows/release.yml/badge.svg)](https://github.com/YOUR_USERNAME/libzmq-native/actions/workflows/release.yml)
```

## Manual Workflow Dispatch

### Building with Custom Versions
1. Go to Actions → Build libzmq Native Libraries
2. Click "Run workflow"
3. Select branch
4. (Optional) Enter custom libzmq version
5. (Optional) Enter custom libsodium version
6. Click "Run workflow"

### Creating Manual Release
1. Ensure all changes are committed
2. Create and push a tag:
   ```bash
   git tag -a v1.0.0 -m "Release v1.0.0"
   git push origin v1.0.0
   ```
3. Or use workflow dispatch:
   - Go to Actions → Release libzmq Native Libraries
   - Click "Run workflow"
   - Enter tag name (e.g., v1.0.0)
   - Click "Run workflow"

## Caching Strategy

### Windows
- **Cache:** vcpkg installed packages
- **Key:** `${{ runner.os }}-vcpkg-${{ libsodium_version }}`
- **Benefit:** Faster libsodium installation via vcpkg

### Linux
- **Cache:** libsodium source build
- **Key:** `${{ runner.os }}-libsodium-${{ libsodium_version }}`
- **Benefit:** Skip libsodium compilation on repeat builds

### macOS (Intel)
- **Cache:** libsodium source build
- **Key:** `${{ runner.os }}-libsodium-x86_64-${{ libsodium_version }}`
- **Benefit:** Skip libsodium compilation on repeat builds

### macOS (ARM64)
- **Cache:** libsodium source build
- **Key:** `${{ runner.os }}-libsodium-arm64-${{ libsodium_version }}`
- **Benefit:** Skip libsodium compilation on repeat builds

## Environment Variables

- `ARTIFACT_RETENTION_DAYS`: 30 days (default)

## Permissions

### build.yml
- `contents: read` (default)

### release.yml
- `contents: write` (required for release creation)

### pr-check.yml
- `contents: read` (default)

## Troubleshooting

### Build Failures

**Windows:**
- Check vcpkg installation
- Verify Visual Studio Build Tools
- Check PowerShell execution policy

**Linux:**
- Verify build dependencies installed
- Check compiler version (gcc/g++)
- Verify CMake version

**macOS:**
- Check Xcode Command Line Tools
- Verify Homebrew packages
- Check architecture-specific settings

### Cache Issues

Clear cache manually:
1. Go to repository Settings → Actions → Caches
2. Delete specific cache entries
3. Re-run workflow

### Release Failures

Common issues:
- Tag already exists: Delete and recreate tag
- Permission denied: Check repository settings → Actions → Workflow permissions
- Missing artifacts: Check build.yml workflow completed successfully

## Integration with Build Scripts

Workflows call platform-specific build scripts:

```yaml
Windows:  build-scripts/windows/build.ps1
Linux:    build-scripts/linux/build.sh
macOS:    build-scripts/macos/build.sh [arch]
```

All build scripts:
- Accept version parameters
- Build libsodium (statically linked)
- Build libzmq with libsodium
- Verify binaries
- Copy to `dist/[platform]/` directory
- Generate build metadata

## Version Management

Version information is stored in `VERSION` file:
```
LIBZMQ_VERSION=4.3.5
LIBSODIUM_VERSION=1.0.19
```

Manual override available via workflow dispatch inputs.
