# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [4.3.5] - 2025-12-14

### Added
- Initial release with pre-built binaries
- Support for Windows x64
- Support for Linux x64
- Support for macOS x64 (Intel)
- Support for macOS ARM64 (Apple Silicon)
- Automated GitHub Actions CI/CD pipeline
- Automatic GitHub Release on tag push

### Features
- libzmq 4.3.5 with CURVE encryption support
- libsodium 1.0.20 statically linked
- Cross-platform build scripts
- Single-file distribution (no external dependencies)

### Build Improvements
- Cross-compilation support for macOS (ARM64 to x86_64)
- CMake policy compatibility for older projects
- Automated artifact packaging and checksums

## [Unreleased]

### Planned
- Linux ARM64 support
- Windows x86 (32-bit) support
- Universal binary for macOS
